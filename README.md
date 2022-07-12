<?php

namespace App\Core\Toshiba\Http\Controllers;

use App\Core\Toshiba\Actions\GetSelectedInquiriesAction;
use App\Core\Toshiba\Actions\GetToshibaStatusAction;
use App\Core\Toshiba\Actions\GetServiceLinksAction;
use App\Core\Toshiba\Actions\SendDataToToshibaAction;
use App\Core\Toshiba\Actions\SendImproveDataAction;
use App\Core\Toshiba\Http\Requests\ShowDiseaseRiskGraphRequest;
use App\Core\Toshiba\Tasks\GetDiseaseRiskTask;
use App\Core\Toshiba\Actions\SetTargetRiskAction;
use App\Core\Toshiba\Actions\ShowDiseaseRiskAction;
use App\Core\Toshiba\Actions\UnconnectHealthCheckDataAction;
use App\Core\Toshiba\Http\Requests\PostImproveRequest;
use App\Core\Toshiba\Http\Requests\ShowDetailImproveRiskRequest;
use App\Core\Toshiba\Models\ImproveHC;
use App\Http\Controllers\Controller;
use Auth;
use Illuminate\Http\Response;
use Illuminate\Http\Request;
use Redis;

class ToshibaController extends Controller
{
    /**
     * Handle status top webview.
     *
     * @return \Illuminate\Http\Response
     */
    public function status()
    {
        
        // dd($this->numberPerfect(-10), $this->numberPerfect(6), $this->numberPerfect(7), $this->numberPerfect(13), $this->numberPerfect(28));
        dd($this->calculatorFactorial(10), $this->calculatorFactorial(5));
        

        [$id, $idNew, $status, $date] = app(GetToshibaStatusAction::class)->handle();
        $params['medical_checkup_app_id'] = $id;
        switch ($status) {
            case config('toshiba.toshiba_top_status.is_not_have_toshiba_risk_prediction_data'):
                return redirect(route('toshiba.service.linkage', $params));
            case config('toshiba.toshiba_top_status.is_not_new_medical_checkup_data_exist'):
                return redirect(route('toshiba.disease_risk', $params));
            default:
                $params['id_new']   = $idNew;
                $params['date_new'] = $date;
                return redirect(route('toshiba.disease_risk', $params));
        }
    }

    /**
     * Check number perfect
     *
     * @param int $number Number
     *
     * @return boolean
     */
    public function numberPerfect(int $number) {
        $sum = 0;

        for ($i = 1; $i <= $number / 2; $i++) {
           $sum = ($number % $i == 0) ? $sum + $i : $sum; 
        }

        if ($number == $sum) {
            return true;
        }

        return false;
    }

    /**
     * Calculate factorial number.
     *
     * @param int $number Number
     *
     * @return int
     */
    public function calculatorFactorial(int $number) {
        $result = 1;

        for ($i = $number; $i > 0; $i--) {
            $result = $result * $i;
        }

        return $result;
    }

    /**
     * Get disease risk webview.
     *
     * @param int $medicalCheckupAppId Medical checkup app id
     *
     * @return \Illuminate\Http\Response
     */
    public function showDiseaseRisk($medicalCheckupAppId)
    {
        try {
            Redis::del('inquiry_answers' . Auth::user()->id);
            $data = app(ShowDiseaseRiskAction::class)->handle($medicalCheckupAppId);
            return view('webviews.toshiba.disease_risk', ['data' => $data]);
        } catch (\Exception $e) {
            \Log::error($e);
            return $this->responseError(trans('toshiba.dialog_error.title'), Response::HTTP_BAD_REQUEST);
        }
    }

    /**
     * Linkage user with toshiba webview.
     *
     * @param string $medicalCheckupsAppId medicalCheckupsAppId
     *
     * @return \App\Http\Controllers\Illuminate\Http\Response
     */
    public function connect($medicalCheckupsAppId)
    {
        $params['medical_checkup_app_id'] = $medicalCheckupsAppId;
        app(SendDataToToshibaAction::class)->handle($medicalCheckupsAppId);
        return redirect(route('toshiba.disease_risk', $params));
    }

    /**
     * Get service linkage webview.
     *
     * @param int $medicalCheckupsAppId medicalCheckupsAppId
     *
     * @return \Illuminate\Http\Response
     */
    public function getServiceLinkage($medicalCheckupsAppId)
    {
        try {
            $data = app(GetServiceLinksAction::class)->handle($medicalCheckupsAppId);
            return view('webviews.toshiba.service_linkage', ["data" => $data]);
        } catch (\Exception $e) {
            \Log::error($e);
            return $this->responseError(trans('toshiba.dialog_error.title'), Response::HTTP_BAD_REQUEST);
        }
    }

    /**
     * Get disease risk graph webview.
     *
     * @param ShowDiseaseRiskGraphRequest $request Request
     *
     * @return \Illuminate\Http\Response
     */
    public function showDiseaseRiskGraph(ShowDiseaseRiskGraphRequest $request)
    {
        try {
            $diseaseRisks = app(GetDiseaseRiskTask::class)->handle($request);
            return view('webviews.toshiba.disease_risk_graph', ['disease_risks' => $diseaseRisks]);
        } catch (\Exception $e) {
            \Log::error($e);
            return $this->responseError(trans('toshiba.dialog_error.title'), Response::HTTP_BAD_REQUEST);
        }
    }

    /**
     * Set target webview.
     *
     * @param string $type            improve type
     * @param string $diseaseRiskHCId diseaseRiskHCId
     *
     * @return \Illuminate\Http\Response
     */
    public function setTarget($type, $diseaseRiskHCId)
    {
        try {
            $data = app(SetTargetRiskAction::class)->handle($diseaseRiskHCId, $type);
            switch ($type) {
                case ImproveHC::TYPE_IMPROVE_WEIGHT:
                    return view('webviews.toshiba.set_target_weight', ['data' => $data]);
                default:
                    return view('webviews.toshiba.set_target_risk', ['data' => $data]);
            }
        } catch (\Exception $e) {
            \Log::error($e);
            return $this->responseError(trans('toshiba.dialog_error.title'), Response::HTTP_BAD_REQUEST);
        }
    }

    /**
     * Unconnect health check data.
     *
     * @param int $medicalCheckupsAppId MedicalCheckupsAppId
     *
     * @return \Illuminate\Http\Response
     */
    public function unconnectHC($medicalCheckupsAppId)
    {
        $params['medical_checkup_app_id'] = $medicalCheckupsAppId;
        try {
            app(UnconnectHealthCheckDataAction::class)->handle($medicalCheckupsAppId);

            return $this->responseSuccess(['url' => route('toshiba.service.linkage', $params)], Response::HTTP_OK);
        } catch (\Exception $e) {
            \Log::error($e);
        }

        return $this->responseError(trans('messages.server_error'), Response::HTTP_INTERNAL_SERVER_ERROR);
    }

    /**
     * Get UI simulation.
     *
     * @return \Illuminate\Http\Response
     */
    public function getUISimulation()
    {
        $inquiry = request()->get('inquiry');
        Redis::set('inquiry_answers' . Auth::user()->id, json_encode($inquiry));

        return view('webviews.toshiba.simulation', ['inquiry' => $inquiry]);
    }

    /**
     * Get UI 11 health check questions.
     *
     * @param \Illuminate\Http\Request $request request
     *
     * @return \Illuminate\Http\Response
     */
    public function getHealthCheckQuestion(Request $request)
    {
        $selectedInquiries = app(GetSelectedInquiriesAction::class)->handle($request->medical_checkup_app_id);
        return view('webviews.toshiba.health_check_question', ['selectedInquiries' => $selectedInquiries]);
    }

    /**
     * Send data to toshiba, AE and get improved result.
     *
     * @param \Illuminate\Http\Request $request request
     *
     * @return \Illuminate\Http\Response
     */
    public function improve(PostImproveRequest $request)
    {
        try {
            $data = app(SendImproveDataAction::class)->handle($request);
            switch ($request->improve_type) {
                case ImproveHC::TYPE_IMPROVE_WEIGHT:
                    return view('webviews.toshiba.improve_weight', ['data' => $data]);
                default:
                    return view('webviews.toshiba.improve_risk', ['data' => $data]);
            }
        } catch (\Exception $e) {
            \Log::error($e);
            return $this->responseError(trans('toshiba.dialog_error.title'), Response::HTTP_BAD_REQUEST);
        }
    }
}

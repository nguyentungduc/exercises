## EX-1

1. Giai thừa của 1 số nguyên dương n.
```bash
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
```
## EX-2

2. Cho định nghĩa sau: số hoàn hảo là số có giá trị bằng tổng các ước nhỏ hơn nó.
 Ví dụ: 6=1+2+3; 28=1+2+4+7+14.
Viết mã giả mô tả thuật toán kiểm tra 1 số n cho trước có phải là số hoàn hảo hay không.
```bash
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
```
## EX-3

3. Với chuỗi cho trước có độ dài lớn hơn 1, viết mã giả thể hiện thuật toán xác định vị trí của 1 kí tự trong chuỗi sao cho khi xoá kí tự đó đi thì chuỗi còn lại là nhỏ nhất có thể.
Ví dụ: chuỗi `231` sau khi xoá đi 1 kí tự có thể trở thành `31`, `21`, `23`. Chuỗi nhỏ nhất là `21`, đáp án đúng là xoá đi kí tự `3` trong chuỗi ban đầu.
```bash
    /**
     * Get value remove index of string min.
     *
     * @param String $number numberString
     *
     * @return array
    */
    public function getValueRemoveIndexOfStringMin(string $numberString) {
        $result = [
            'min'   => substr_replace($numberString, '', 0, 1),
            'index' => 0,
            'value' => $numberString[0],
        ];

        for ($i = 1; $i < strlen($numberString); $i++) {
            if ($result['min'] > substr_replace($numberString, '', $i, 1)) {
                $result = [
                    'min'   => substr_replace($numberString, '', $i, 1),
                    'index' => $i,
                    'value' => $numberString[$i],
                ];
            };
        }

        return $result;
    }
```
## EX-4

4. Bob cầm n đồng đi mua kẹo. Giá của 1 viên kẹo là c đồng. Cửa hàng lại có chương trình khuyến mại cứ m tờ giấy gói kẹo thì đổi được 1 viên kẹo mới. Viết mã giả thể hiện thuật toán xác định tổng số kẹo Bob có thể mua được từ các tham số n, c, m như đã mô tả ở trên.
 - n=10, c=2, m=5: Bob mua được 5 viên kẹo, sau đó đổi 5 tờ giấy gói lấy 1 viên kẹo nữa, tổng cộng đáp án là 6.
 - n=12, c=4, m=4: Bob mua được 3 viên kẹo, số giấy gói kẹo không đủ đổi thêm nữa, đáp án cuối cùng là 3.
 - n=6, c=2, m=2: Bob mua được 3 viên kẹo, lấy 2 trong 3 tờ giấy gói kẹo đổi được thêm 1 viên. Sau đó dùng tiếp 1 tờ giấy gói dư ở lần đổi thứ nhất + tờ giấy gói của viên kẹo ở lần đổi thứ 2, đổi được thêm 1 viên kẹo nữa. Đáp án tổng cộng là 5.

```bash
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
```

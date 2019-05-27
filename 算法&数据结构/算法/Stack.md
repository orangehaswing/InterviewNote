# Stack

## 计算器算法题

LeetCode 224. Basic Calculator

字符串中包含括号，+ - 没有空格和负数

```
Input: "1 + 1"
Output: 2
Input: "1 + 1"
Output: 2
Input: "(1+(4+5+2)-3)+(6+8)"
Output: 23
```

res：输出结果 number：保存加数值 sign 保存这次的运算符

对2-1+(4+5)举例：'2' -> num = 2; '-' -> res = 2, sigin = -1; '+' -> res = 2+ (-)*1:把前面一项结果计算出来；遇到括号，先保存res和sign值到栈，再计算括号内的值。最后从栈取出与括号内的值相加

    public int calculate(String s) {
    	char[] c = s.toCharArray();
        Stack<Integer> stack = new Stack<>();
        int number = 0;
        int res = 0;
        int sign = 1;
        for (int i = 0; i < c.length; i++) {
            if (Character.isDigit(c[i])){
                number = 10*number + (int)(c[i] - '0');
            }else if (c[i] == '+'){
                res += sign *number;
                number = 0;
                sign = 1;
            }else if (c[i] == '-'){
                res += sign *number;
                number = 0;
                sign = -1;
            }else if (c[i] == '('){
                stack.push(res);
                stack.push(sign);
                sign = 1;
                res = 0;
            }else if (c[i] == ')'){
                res += sign*number;
                number = 0;
                res = stack.pop()*res + stack.pop();
            }
        }
    
        if (number != 0){
            res += sign*number;
        }
        return res;
    }
LeetCode 227 Basic Calculator II

实现计算器，包含非负数，+ - * / 和空格，没有括号

```
Input: "3+2*2"
Output: 7
Input: " 3/2 "
Output: 1
Input: " 3+5 / 2 "
Output: 5
```

首先保存sign符号，当number指向下一个值，如果sign为* 或/ ，从栈中取出前一个数字与number做运算然后压入栈中。否则放入栈中(‘-’号要把number取负号)，求最终结果的时候，从栈中取出所有数字相加

```
public int calculate(String s) {
        char[] c = s.toCharArray();
        Stack<Integer> stack = new Stack<>();
        int number = 0;
        char sign = '+';
        for (int i = 0; i < c.length; i++) {
            if (Character.isDigit(c[i])){
                number = number*10 + (int)(c[i] - '0');
            }

            if (!Character.isDigit(c[i]) && c[i] != ' ' || i == c.length-1){
                if (sign == '+'){
                    stack.push(number);
                }if (sign == '-'){
                    stack.push(-number);
                }
                if (sign == '*'){
                    stack.push(stack.pop()*number);
                }
                if (sign == '/'){
                    stack.push(stack.pop()/number);
                }
                sign = c[i];
                number = 0;
            }
        }

        int res = 0;
        for (int i:stack) {
            res += i;
        }

        return res;
    }
```

LeetCode 772 Basic Calculator III

字符串表达式包含非负数，+ - * / 括号 空格

```
"1 + 1" = 2
" 6-4 / 2 " = 4
"2*(5+5*2)/3+(6/2+8)" = 21
"(2+6* 3+5- (3*14/7+2)*5)+3"=-12
```



```
public int calculate(String s) {
        s = s.replaceAll(" ", "");
        if (s.length() == 0) return 0;
        Stack<Integer> stack = new Stack<>();
        char sign = '+';
        int res = 0, pre = 0, i = 0;
        while (i < s.length()) {
            char ch = s.charAt(i);
            //consecutive digits as a number, save in `pre`
            if (Character.isDigit(ch)) {
                pre = pre*10+(ch-'0');
            }
            //recursively calculate results in parentheses
            if (ch == '(') {
                int j = findClosing(s.substring(i));
                pre = calculate(s.substring(i+1, i+j));
                i += j;
            }
            //for new signs, calculate with existing number/sign, then update number/sign
            if (i == s.length()-1 || !Character.isDigit(ch)) {
                switch (sign) {
                    case '+':
                        stack.push(pre); break;
                    case '-':
                        stack.push(-pre); break;
                    case '*':
                        stack.push(stack.pop()*pre); break;
                    case '/':
                        stack.push(stack.pop()/pre); break;
                }
                pre = 0;
                sign = ch;
            } 
            i++;
        }
        while (!stack.isEmpty()) res += stack.pop();
        return res;
    }
    
    //Eliminate all "()" pairs, calculate the result in between and save in `pre`
    private int findClosing(String s) {
        int level = 0, i = 0;
        for (i = 0; i < s.length(); i++) {
            if (s.charAt(i) == '(') level++;
            else if (s.charAt(i) == ')') {
                level--;
                if (level == 0) break;
            } else continue;
        }
        return i;
    }
```

LeetCode 770 Basic Calculator IV



## 字符串编码解码

**LeetCode 394 Decode String**

给出字符串编码形式，求解码。保证字符串有效

```
s = "3[a]2[bc]", return "aaabcbc".
s = "3[a2[c]]", return "accaccacc".
s = "2[abc]3[cd]ef", return "abcabccdcdcdef".
```

用两个Stack，其中一个保存数字，另一个保存字符串结果

```
	public String decodeString(String s) {
        Stack<Integer> stack = new Stack<>();
        Stack<String> res = new Stack<>();

        res.push("");
        int i = 0;
        while (i < s.length()) {
            char c = s.charAt(i);
			//保存数字
            if (c >= '0' && c <= '9'){
                int start = i;
                while (s.charAt(i+1) >= '0' && s.charAt(i+1) <= '9') i++;

                int num = Integer.parseInt(s.substring(start,i+1));
                stack.push(num);
            }else if (c == '['){
                res.push("");
            // 对‘]’之前的string 和 num做处理，并且将结果与res拼接。
            }else if (c == ']'){
                String str = res.pop();
                StringBuilder sb = new StringBuilder();
                int times = stack.pop();
                for (int j = 0; j < times; j++) {
                    sb.append(str);
                }
                res.push(res.pop()+sb.toString());
            //在‘[’ 和‘]’之间的字符串相加
            }else {
                res.push(res.pop() + c);
            }
            i++;
        }

        return res.pop();
    }
```

## 括号

LeetCode 20 Valid Parentheses

给定字符串，只包含'()','[]','{}',求是否成对出现

```
Input: "()"
Output: true
Input: "()[]{}"
Output: true
Input: "(]"
Output: false
```

用stack实现，出现'(','[','{'时，将相对应的反括号压入栈

```
 	public boolean isValid(String s) {
        Stack<Character> stack = new Stack<>();
        char[] c = s.toCharArray();
        for (char a:c){
            if (a == '('){
                stack.push(')');
            }else if (a == '{'){
                stack.push('}');
            }else if (a == '['){
                stack.push(']');
            } else if (stack.isEmpty() || stack.pop() != a){
                return false;
            }
        }
        return stack.isEmpty();
    }
```












































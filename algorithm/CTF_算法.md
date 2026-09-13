## 一、base64解密脚本
题目：
~~~Python
import string

def encode(string,string2):
    tmp_str = str()
    ret = str()
    bit_string_str = string.encode()
    remain = len( string ) % 3
    remain_str = str()
    for char in bit_string_str:
        b_char = (bin(char)[2:])
        b_char = '0'*(8-len(b_char)) + b_char
        tmp_str += b_char
    for i in range(len(tmp_str)//6):
        temp_nub = int(tmp_str[i*6:6*(i+1)],2)
        ret += string2[temp_nub]
    if remain==2:
        remain_str = tmp_str[-4:] + '0'*2
        temp_nub = int(remain_str,2)
        ret += string2[temp_nub] + "="
    elif remain==1:
        remain_str = tmp_str[-2:] + '0'*4
        temp_nub = int(remain_str,2)
        ret += string2[temp_nub] + "="*2
    return ret.replace("=","")

res = encode(input(),string.ascii_uppercase+string.ascii_lowercase+string.digits+'+/')

if res == "TlNTQ1RGe2Jhc2U2NCEhfQ":
    print("good!")
else:
    print("bad!")
~~~
**注意**：base64串的长度需为4的整数倍，所以在解码的时候需要用“=”来补齐
解题code：
~~~Python
import base64  
flag = "TlNTQ1RGe2Jhc2U2NCEhfQ=="  
a = base64.b64decode(flag).decode("utf-8")  
print(a)
~~~

## 二、凯撒解密
~~~python
def caesar_decrypt(cipher_text: str, shift: int) -> str:
    """
    凯撒密码解密
    :param cipher_text: 密文字符串
    :param shift: 移位值（加密时右移多少，解密就左移多少）
    :return: 明文
    """
    plain_text = ""
    shift = shift % 26  # 超过26取模，防止溢出
    for char in cipher_text:
        if char.isupper():
            # 大写字母 A-Z
            plain_text += chr((ord(char) - ord('A') - shift) % 26 + ord('A'))
        elif char.islower():
            # 小写字母 a-z
            plain_text += chr((ord(char) - ord('a') - shift) % 26 + ord('a'))
        else:
            # 非字母直接保留
            plain_text += char
    return plain_text


def brute_force_caesar(cipher_text: str):
    """凯撒暴力破解，遍历所有26种移位"""
    print("===== 凯撒密码暴力破解结果 =====")
    for s in range(26):
        res = caesar_decrypt(cipher_text, s)
        print(f"shift={s:2d} | {res}")


if __name__ == "__main__":
    # ========== 使用示例 ==========
    cipher = "vlhjh vklyhu vloob krfnhb"  # 密文，hello world 移位3加密
    print("密文：", cipher)
    print("\n开始暴力破解：")
    brute_force_caesar(cipher)
~~~

## 三、转ASCll / 转字符串
转ASCll用ord()
~~~python
def str_to_ascii(s: str) -> list[int]:
    return [ord(c) for c in s]

if __name__ == "__main__":
    text = "Hello"
    ascii_list = str_to_ascii(text)
    print(f"字符串：{text}")
    print(f"ASCII码列表：{ascii_list}")
    # 空格分隔输出
    print(f"ASCII(空格隔开): {' '.join(map(str, ascii_list))}")
~~~
转字符串用chr()
~~~python
def ascii_to_str(ascii_arr: list[int]) -> str:
    return ''.join([chr(num) for num in ascii_arr])

if __name__ == "__main__":
    nums = [72, 101, 108, 108, 111]
    res = ascii_to_str(nums)
    print(res) # Hello
~~~

## 四、大整数
~~~python
from Crypto.Util.number import *

a = 11515195063862318899931685488813747395775516287289682636499965282714637259206269
b = long_to_bytes(a)
print(b)
~~~
也可以用bytes_to_long把消息转换成整数方便计算

## 五、异或
**异或是对单字节做运算**
- 将label和13进行异或
~~~python
s = "label"
result = [ord(c) ^ 13 for c in s]
print("每个字符异或后的数字：", result)

# 转回字符串
xor_str = ''.join([chr(ord(c) ^ 13) for c in s])
print("异或后的字符串：", xor_str)
~~~

- 异或特性：
交换律：A ⊕ B = B ⊕ A  
结合律：A ⊕ (B ⊕ C) = (A ⊕ B) ⊕ C  
单位元：A ⊕ 0 = A  
自逆元：A ⊕ A = 0
~~~python
KEY1 = a6c8b6733c9b22de7bc0253266a3867df55acde8635e19c73313  
KEY2 ^ KEY1 = 37dcb292030faa90d07eec17e3b1c6d8daf94c35d4c9191a5e1e  
KEY2 ^ KEY3 = c1545756687e7573db23aa1c3452a098b71a7fbf0fddddde5fc1  
FLAG ^ KEY1 ^ KEY3 ^ KEY2 = 04ee9855208a2cd59091d04767ae47963170d1660df7f56f5faf
~~~
解
~~~python
from Crypto.Util.number import long_to_bytes, bytes_to_long

a1 = 0xa6c8b6733c9b22de7bc0253266a3867df55acde8635e19c73313
a3 = 0xc1545756687e7573db23aa1c3452a098b71a7fbf0fddddde5fc1
a4 = 0x04ee9855208a2cd59091d04767ae47963170d1660df7f56f5faf

f = a1 ^ a3 ^ a4
ff = long_to_bytes(f)
print(ff)
~~~

- 使用异或特性进行单字节爆破
题目：73626960647f6b206821204f21254f7d694f7624662065622127234f726927756d
解题：
1）
暴力破解：
~~~python
from Crypto.Util.number import long_to_bytes, bytes_to_long

a = 0x73626960647f6b206821204f21254f7d694f7624662065622127234f726927756d
cipher_bytes = long_to_bytes(a) # 把大整数还原成一串字节

# 遍历所有单字节密钥0~255
for key in range(256):
    # 每个字节单独和key异或
    plain_bytes = bytes( [byte ^ key for byte in cipher_bytes] )
    try:
        plaintext = plain_bytes.decode('ascii')
        print(f"key={key:3d} | {plaintext}")
    except:
        # 无法转ascii的乱码直接跳过不打印
        pass
~~~
2）
由于已知明文是以crypto开头，所以：
~~~python
input_str = bytes.fromhex('73626960647f6b206821204f21254f7d694f7624662065622127234f726927756d')

key = input_str[0] ^ ord('c')
print(''.join(chr(c ^ key) for c in input_str))
~~~
bytes.fromhex：把一串十六进制文本，转换成 bytes 字节对象
bytes.hex：bytes 转回十六进制字符串
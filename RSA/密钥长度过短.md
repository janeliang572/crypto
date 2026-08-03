## 二、p 、q长度过短
题目：
~~~Python
from Crypto.Util.number import getPrime, inverse, bytes_to_long, GCD

FLAG = b"NSSCTF{*******}"

p = getPrime(496)    
q = getPrime(16)     
N = p * q
phi = (p - 1) * (q - 1)

for k_candidate in range(2, 200):
    e_candidate = 65537 * k_candidate + 1
    if GCD(e_candidate, phi) == 1:
        k = k_candidate
        e = e_candidate
        break

d = inverse(e, phi)
m = bytes_to_long(FLAG)
c = pow(m, e, N)


print(f"N = {N}")
print(f"c = {c}")
print(f"e_mod_65537 = {e % 65537}")   
print(f"PHI_mod_65537 = {phi % 65537}")

# N = 7766768698831459057447624753799597048605145237159899787468949300915452509464565099496679099145810975822175634998097262892579678554704760323902554901001931
# c = 3032034114991327495494398063970483186189941361342590220935258419146172566936781482456512156238329348954750763279841189333666924656636066281016492074315344
# e mod 65537 = 1
# PHI_mod_65537 = 43577
~~~
首先看到q只有16位，属于小素数，所以直接暴力破解：
~~~Python
from Crypto.Util.number import isPrime, long_to_bytes

N = 7766768698831459057447624753799597048605145237159899787468949300915452509464565099496679099145810975822175634998097262892579678554704760323902554901001931
c = 3032034114991327495494398063970483186189941361342590220935258419146172566936781482456512156238329348954750763279841189333666924656636066281016492074315344

# q 是16bit质数：2^15 ~ 2^16-1
start = 1 << 15
end = (1 << 16) - 1

q_found = None
for q in range(start, end+1):
    if isPrime(q):
        if N % q == 0:
            q_found = q
            break

p = N // q_found
print(f"p = {p}")
print(f"q = {q_found}")

phi = (p-1)*(q-1)

# 还原e（可选，验证用）
# 遍历k 2~199，e=65537k+1,gcd(e,phi)=1
from Crypto.Util.number import GCD, inverse
e = None
for k_candidate in range(2, 200):
    e_candidate = 65537 * k_candidate + 1
    if GCD(e_candidate, phi) == 1:
        e = e_candidate
        break
print(f"e = {e}")

d = inverse(e, phi)
m = pow(c, d, N)
flag = long_to_bytes(m)
print("FLAG:", flag)
~~~


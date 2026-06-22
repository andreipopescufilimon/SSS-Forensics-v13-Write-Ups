First I check the png metadata, which had this comment:
```ccp
    	X3lyM3Zle1NTUw==
```
Which is in base64 and it says 

```ccp
    	_yr3ve{SSS
```
And flipped is 
```ccp
    	SSS{ev3ry_
```
After that the artist name is:
```ccp
      X3NldjQzbF90YzR0bjBj
```
Also in base64
```ccp
      _sev43l_tc4tn0c
```
And also flipped
```ccp
     c0nt4ct_l34ves_
```
After i have checked the metadata, i went to see the bytes in the photo. Thats where i found out that there are some extra hex values after the IEND value, using the pngchecker in ubuntu WLS.
The Hex value is:
```ccp
61 5f 74 72 34 63 65 5f 73 6f 6d 33 77 48 33 72 65 7d
```
And translated is:
```cpp
a_tr4ce_som3wH3re}
```

So final Flag is :
```cpp
SSS{ev3ry_c0nt4ct_l34ves_a_tr4ce_som3wH3re}
```

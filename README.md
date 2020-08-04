# ga4d
Geometric Algebra for D

D has incredible metaprogramming features which can be leveraged to generate code for performing efficient Geometric Algebra computations.
Here, 'efficient' means both avoiding creating temporaries and performing needless computations.

## Preview

### Euclidean 3D

##### Generated code to compute the 'wedge' (aka 'outer') product of vectors a and b, yielding bivector a^b
```
ab[0] = (a[0]*b[1])-(a[1]*b[0]);
ab[1] = (a[0]*b[2])-(a[2]*b[0]);
ab[2] = (a[1]*b[2])-(a[2]*b[1]);
```

#### Regressive product
A straightforward implementation of the regressive product in G3 might look like:
```
auto regressiveProduct(A, B)(A a, B b)
{
  return ((a * _i3) ^ (b * _i3)) * i3; // i3 is the unit pseudoscalar of G3 and _i3 = -i3
}
```
but one should be concerned about the creation of temporaries and needless multiplies by -1 and 1.

##### Generated code to compute the regressive product of bivectors a and b, yielding vector m
```
m[0] = -(((((a[1]*(-1)))*(-(b[0]*(-1))))-((-(a[0]*(-1)))*((b[1]*(-1)))))*(1));
m[1] = ((((-(a[2]*(-1)))*(-(b[0]*(-1))))-((-(a[0]*(-1)))*(-(b[2]*(-1)))))*(1));
m[2] -((((-(a[2]*(-1)))*((b[1]*(-1))))-(((a[1]*(-1)))*(-(b[2]*(-1)))))*(1));
```
This looks gross. However, ldc generates the following code with the -O flag set:
```
movq    rax, xmm2
movd    xmm2, eax
shr     rax, 32
movd    xmm4, eax
movq    rax, xmm0
movd    xmm0, eax
shr     rax, 32
movd    xmm5, eax
movaps  xmm6, xmm1
mulss   xmm6, xmm2
mulss   xmm2, xmm5
mulss   xmm1, xmm4
mulss   xmm4, xmm0
subss   xmm2, xmm4
movaps  xmm4, xmmword ptr [rip + .LCPI0_0]
xorps   xmm2, xmm4
movss   dword ptr [rsp - 8], xmm2
mulss   xmm0, xmm3
subss   xmm0, xmm6
movss   dword ptr [rsp - 4], xmm0
mulss   xmm3, xmm5
subss   xmm1, xmm3
xorps   xmm1, xmm4
movsd   xmm0, qword ptr [rsp - 8]
ret
```
A similar result is obtained using dmd.

### Quaternions

##### Generated code to multiply 2 quaternions q and r
```
qr[0] = (q[0]*r[0])-(q[1]*r[1])-(q[2]*r[2])-(q[3]*r[3]);
qr[1] = (q[0]*r[1])+(q[1]*r[0])+(q[2]*r[3])-(q[3]*r[2]);
qr[2] = (q[0]*r[2])-(q[1]*r[3])+(q[2]*r[0])+(q[3]*r[1]);
qr[3] = (q[0]*r[3])+(q[1]*r[2])-(q[2]*r[1])+(q[3]*r[0]);
```

##### Generated code to multiply quaternion q by quaternion v where v is 'pure imaginary' in that it has no scalar part

```
qv[0] = -(q[1]*v[0])-(q[2]*v[1])-(q[3]*v[2]);
qv[1] = (q[0]*v[0])+(q[2]*v[2])-(q[3]*v[1]);
qv[2] = (q[0]*v[1])-(q[1]*v[2])+(q[3]*v[0]);
qv[3] = (q[0]*v[2])+(q[1]*v[1])-(q[2]*v[0]);
```

##### Let q and v be as above, and let p = [q[0], -q[1], -q[2], -q[3]], i.e., p == the Clifford conjugate of q
```
qvp[0] = ((-(q[1]*v[0])-(q[2]*v[1])-(q[3]*v[2]))*(q[0]))-(((q[0]*v[0])+(q[2]*v[2])-(q[3]*v[1]))*(-q[1]))-(((q[0]*v[1])-(q[1]*v[2])+(q[3]*v[0]))*(-q[2]))-(((q[0]*v[2])+(q[1]*v[1])-(q[2]*v[0]))*(-q[3]));
qvp[1] = ((-(q[1]*v[0])-(q[2]*v[1])-(q[3]*v[2]))*(-q[1]))+(((q[0]*v[0])+(q[2]*v[2])-(q[3]*v[1]))*(q[0]))+(((q[0]*v[1])-(q[1]*v[2])+(q[3]*v[0]))*(-q[3]))-(((q[0]*v[2])+(q[1]*v[1])-(q[2]*v[0]))*(-q[2]));
qvp[2] = ((-(q[1]*v[0])-(q[2]*v[1])-(q[3]*v[2]))*(-q[1]))+(((q[0]*v[0])+(q[2]*v[2])-(q[3]*v[1]))*(q[0]))+(((q[0]*v[1])-(q[1]*v[2])+(q[3]*v[0]))*(-q[3]))-(((q[0]*v[2])+(q[1]*v[1])-(q[2]*v[0]))*(-q[2]));
qvp[3] = ((-(q[1]*v[0])-(q[2]*v[1])-(q[3]*v[2]))*(-q[3]))+(((q[0]*v[0])+(q[2]*v[2])-(q[3]*v[1]))*(-q[2]))-(((q[0]*v[1])-(q[1]*v[2])+(q[3]*v[0]))*(-q[1]))+(((q[0]*v[2])+(q[1]*v[1])-(q[2]*v[0]))*(q[0]));
```
This computation comes up when we want to rotate vectors using normalized quaternions and their conjugates.
It is worth noting that qvp[0] can only ever be = 0. Expanding, 
```
qvp[0] =
-(q[1] * v[0] * q[0]) - (q[2] * v[1] * q[0]) - (q[3] * v[2] * q[0])) +
(q[0] * v[0] * q[1]) + (q[2] * v[2] * q[1]) - (q[3] * v[1] * q[1]) +
(q[0] * v[1] * q[2]) - (q[1] * v[2] * q[2]) + (q[3] * v[0] * q[2]) +
(q[0] * v[2] * q[3]) + (q[1] * v[1] * q[3]) - (q[2] * v[0] * q[3]));
```
i.e., the positive and negative terms cancel. Therefore, a code generator must skip the computation of qvp[0], as well as avoid reserving an array element for it.

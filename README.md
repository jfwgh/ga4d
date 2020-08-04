# ga4d
Geometric Algebra for D

The user is assumed to be familiar with Geometric Algebra (GA) and how it can be used in various areas of computer science.

This project aims to achieve the goal of performing efficient GA computations by leveraging D's powerful metaprogramming features.

Hopefully, we can arrive at a point where a 3D engine can leverage GA without having to tolerate performance shortcomings.

## Generated code preview

### Euclidean 3D -- G<sup>3</sup>

##### Computing the 'wedge' (aka 'outer') product of vectors a and b, yielding bivector a^b
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

##### Computing the regressive product of bivectors a and b, yielding vector m
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

### Quaternions -- G<sup>0,2</sup>
The basis for G<sup>0,2</sup> is { 1, <b>e<sub>1</sub></b>, <b>e<sub>2</sub></b>, <b>e<sub>12</sub></b> } = { 1, <b>i</b>, <b>j</b>, <b>k</b> } .

For quaternion Q, Q[0] refers to the value of the '1' component (that is, the 'scalar part' of Q), Q[1] to the value of component <b>i</b>, etc.

A quaternion with scalar part == 0 is termed 'pure imaginary'.

##### Computing the product of quaternions q and r
```
qr[0] = (q[0]*r[0])-(q[1]*r[1])-(q[2]*r[2])-(q[3]*r[3]);
qr[1] = (q[0]*r[1])+(q[1]*r[0])+(q[2]*r[3])-(q[3]*r[2]);
qr[2] = (q[0]*r[2])-(q[1]*r[3])+(q[2]*r[0])+(q[3]*r[1]);
qr[3] = (q[0]*r[3])+(q[1]*r[2])-(q[2]*r[1])+(q[3]*r[0]);
```

##### Product of q and v where v is pure imaginary
```
qv[0] = -(q[1]*v[0])-(q[2]*v[1])-(q[3]*v[2]);
qv[1] = (q[0]*v[0])+(q[2]*v[2])-(q[3]*v[1]);
qv[2] = (q[0]*v[1])-(q[1]*v[2])+(q[3]*v[0]);
qv[3] = (q[0]*v[2])+(q[1]*v[1])-(q[2]*v[0]);
```
Here, <b>v</b> has 3 components instead of 4; it's still a member of G<sup>0,2</sup> but has a a different _type_ from Q (think 'PureQuaternion' vs. 'Quaternion').

##### Let q and v be as above, and let p = the Clifford conjugate of q
p = [q[0], -q[1], -q[2], -q[3]]
```
qvp[0] = ((-(q[1]*v[0])-(q[2]*v[1])-(q[3]*v[2]))*(q[0]))-(((q[0]*v[0])+(q[2]*v[2])-(q[3]*v[1]))*(-q[1]))-(((q[0]*v[1])-(q[1]*v[2])+(q[3]*v[0]))*(-q[2]))-(((q[0]*v[2])+(q[1]*v[1])-(q[2]*v[0]))*(-q[3]));
qvp[1] = ((-(q[1]*v[0])-(q[2]*v[1])-(q[3]*v[2]))*(-q[1]))+(((q[0]*v[0])+(q[2]*v[2])-(q[3]*v[1]))*(q[0]))+(((q[0]*v[1])-(q[1]*v[2])+(q[3]*v[0]))*(-q[3]))-(((q[0]*v[2])+(q[1]*v[1])-(q[2]*v[0]))*(-q[2]));
qvp[2] = ((-(q[1]*v[0])-(q[2]*v[1])-(q[3]*v[2]))*(-q[1]))+(((q[0]*v[0])+(q[2]*v[2])-(q[3]*v[1]))*(q[0]))+(((q[0]*v[1])-(q[1]*v[2])+(q[3]*v[0]))*(-q[3]))-(((q[0]*v[2])+(q[1]*v[1])-(q[2]*v[0]))*(-q[2]));
qvp[3] = ((-(q[1]*v[0])-(q[2]*v[1])-(q[3]*v[2]))*(-q[3]))+(((q[0]*v[0])+(q[2]*v[2])-(q[3]*v[1]))*(-q[2]))-(((q[0]*v[1])-(q[1]*v[2])+(q[3]*v[0]))*(-q[1]))+(((q[0]*v[2])+(q[1]*v[1])-(q[2]*v[0]))*(q[0]));
```
This computation comes up when we want to rotate vectors using normalized quaternions and their conjugates. In this context, <b>v</b> can be interpreted as a 3D vector.

It is worth noting that qvp[0] can only ever be = 0. Expanding, 
```
qvp[0] =
-(q[1] * v[0] * q[0]) - (q[2] * v[1] * q[0]) - (q[3] * v[2] * q[0])) +
(q[0] * v[0] * q[1]) + (q[2] * v[2] * q[1]) - (q[3] * v[1] * q[1]) +
(q[0] * v[1] * q[2]) - (q[1] * v[2] * q[2]) + (q[3] * v[0] * q[2]) +
(q[0] * v[2] * q[3]) + (q[1] * v[1] * q[3]) - (q[2] * v[0] * q[3]));
```
i.e., the positive and negative terms cancel.

Thus qvp is automatically pure imaginary. Ideally, a code generator would realize this. It would skip the computation of qvp[0] and ensure that qvp is of type PureQuaternion.


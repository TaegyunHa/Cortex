

$y_c = \frac{n}{t} \cdot y$
$w_c = −z$

Near plane:
$y=t$
$z=-n$
$z_ndc=-1$

$$
z = Cz + D = C(-n) + D
$$
Substitue: $C = -\frac{f+n}{f-n}$
$$
= (-\frac{f+n}{f-n})(-n) + D
$$
$$
= n(\frac{f+n}{f-n}) + D
$$
Substitue: $D = -\frac{2fn}{f-n}$
$$
=\frac{n(f+n)}{f-n} - \frac{2fn}{f-n}
$$
$$
=\frac{n(f+n)-2fn}{f-n} = \frac{nf+n^2-2fn}{f-n} = \frac{n(n-f)}{f-n}
$$
Thus
$$
Cz+D = \frac{n(n-f)}{f-n} = -n
$$
### Denominator
4th row of the projection matrix:
$w_c = -z$

> The denominator is just `w_c`, and `w_c` was _defined_ to be `-z`, so on the near plane it must be `n`

Near plane:
$z=-n$

Therefore:
$$
w_c = -z = -(-n) = n
$$
With the derived $z_c$:
$z_c = -n$

$$
z_ndc = \frac{z_c}{w_c} = \frac{-n}{n} = -1
$$

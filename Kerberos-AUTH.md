İstifadəçi username və password yazır. Client tərəfdəki LSASS password‑u götürür və onu NT hash‑ə çevirir. Bu NT hash artıq istifadəçinin UserKey‑idir.

LSASS cari vaxtdan bir timestamp yaradır.

Timestamp Yaradilmasi:

✔ OS‑dən cari vaxtı oxuyur

✔ UTC formatına çevirir

✔ Kerberos formatında saxlayır

Misal: 20260225051432Z

Daha sonra bu **Timestamp** *UserKey* ( userin pass yazanda lsass-in yaratdigi nt hash ) ilə şifrələyərək **Pre‑Authentication** data hazırlayır. Client bu məlumatı AS‑REQ sorğusu kimi Domain Controller‑də yerləşən KDC serverinə göndərir.

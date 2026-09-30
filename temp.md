\[jkitchin/tennessee-eastman-profbraatz]

Bu mimari Gymnasium'a sarılması en kolay seçeneklerden biridir, fakat iki iyileştirme gerekir. Birincisi

integrator interface'ini RHS'den ayırmak; ikincisi stochastic variables'ın solver substeps'ine bağımlı

olmadığını regression test ile doğrulamaktır. Yalnızca mevcut Euler step() üzerinde Gym wrapper

kurmak, hızlı ve tarihsel açıdan tutarlı bir baseline verir; fakat solver araştırması yapmak için yeterli

değildir. 2015 çalışmasının random-number problemi burada özellikle dikkate alınmalıdır







**--------------------------------------------------------------------------------------------------------------------------**







Ben olsam geliştirmeyi şöyle bölerdim

Amaç	Solver	Internal dt	Rol

Canonical RL baseline	Euler	1 s	Ana environment

Solver sensitivity	RK4	1 s	Karşılaştırma

Convergence check	RK4	0.25 veya 0.1 s	Numerical reference

Daha sonra gerekirse	Radau/BDF/LSODA	adaptive	High-accuracy validation





Özellikle üçüncü satır önemli. RK4 1 s ile Euler 1 s farklı çıkarsa hangisinin “doğru” olduğunu tek başına bilmiyoruz. O yüzden RK4'ü 1 s → 0.5 s → 0.25 s → 0.1 s şeklinde küçülterek trajectory'nin converge edip etmediğine bakmak çok daha anlamlı.





**--------------------------------------------------------------------------------------------------------------------------**



**OUR RESEARCH QUESTION**

Can selective state-space models learn control-sufficient representations for reinforcement learning in partially observable nonlinear dynamical systems with long-term dependencies?







**--------------------------------------------------------------------------------------------------------------------------**



Baktım; en kolay ve en temiz yol, 1993 Fortran’ı elle patchlemek değil. Ben sizin yerinizde 2015 temexd\_mod.c kernelini baz alıp yalnızca 1993-compatible bir profil açardım. Çünkü 2015 kodunun içinde zaten hangi revizyonların aktif olacağını seçen bir bit-mask mekanizması var.

En güzel bulgu şu: 2015 kaynak kodunda bit 5 measurement noise ile process disturbance için iki ayrı RNG state kullanıyor; bit 6 ise random disturbance’ların yeniden hesaplanmasını solver çağrısına değil önceden belirlenmiş fiziksel zamana bağlıyor. Bunlar tam olarak istediğiniz solver/reproducibility düzeltmeleri. Ek measurement’lar bit 0, monitoring output’lar bit 1–3, noise kapatma bit 4 ve disturbance scaling bit 7 ile ayrı ayrı açılıyor. 

https://github.com/camaramm/tennessee-eastman-challenge/blob/master/temexd\_mod/temexd\_mod.c



Böylece 2015'in tamamını almıyorsun; özellikle:



bit 0 = OFF   → 42–73 additional measurements yok

bit 1–3 = OFF → privileged monitoring outputs yok

bit 4 = OFF   → measurement noise korunuyor

bit 5 = ON    → separate RNG states

bit 6 = ON    → time-based disturbance recalculation

bit 7 = OFF   → noncanonical disturbance scaling yok




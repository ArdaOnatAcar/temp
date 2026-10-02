**☺	Model free s Model based** 

**☺	Veri üretimi** 

&#x09;**Timeseries timestep (Delta t)**

&#x09;**RL e beslenen vektör zamanı timestepi ve boyutu**

**☺	1993 vs 2015**



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








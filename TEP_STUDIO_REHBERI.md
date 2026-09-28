# TEP Studio 0.2.0 — Türkçe Kurulum ve Kullanım Rehberi

Bu rehber, **TEP Studio 0.2.0** kütüphanesini bu bilgisayardaki mevcut kurulumla çalıştırmak, web arayüzünü kullanmak, komut satırından deney yürütmek ve Python üzerinden simülasyon/veri seti/RL çalışmaları yapmak için hazırlanmıştır.

> TEP Studio bir araştırma ve eğitim simülatörüdür. Gerçek bir tesise bağlı değildir ve gerçek tesis kontrolü için doğrudan kullanılmamalıdır.

## 1. Bu bilgisayardaki kurulum

Kurulum aşağıdaki conda ortamındadır:

| Bileşen | Değer |
|---|---|
| Conda ortamı | `tep` |
| Python | 3.12.14 (64 bit) |
| TEP Studio | 0.2.0 |
| Web arayüzü | Dash + Plotly |
| Native hesaplama | CFFI ile derlenmiş Windows `.pyd` modülü |
| Varsayılan adres | <http://127.0.0.1:8050> |

Kurulumun doğru olduğunu kontrol etmek için:

```powershell
conda run -n tep tep version
conda run -n tep python -m pip check
conda run -n tep python -c "import tep_studio as tep; tep.quickstart()"
```

Başarılı hızlı testin benzeri şu çıktıyı vermesi beklenir:

```text
TEP quickstart: stabilized=True over 8.0 h; ...
```

## 2. Başlatma ve kapatma

### 2.1 Web arayüzünü başlatma

En sağlam yöntem, ortamı etkinleştirmeden `conda run` kullanmaktır:

```powershell
conda run -n tep tep ui
```

Ardından tarayıcıda açın:

```text
http://127.0.0.1:8050
```

Alternatif olarak ortamı etkinleştirebilirsiniz:

```powershell
conda activate tep
tep ui
```

PowerShell `conda activate` komutunu tanımıyorsa bir defaya mahsus:

```powershell
conda init powershell
```

komutunu çalıştırıp PowerShell'i yeniden açın. `conda run -n tep ...` yöntemi bu işleme ihtiyaç duymaz.

### 2.2 Farklı port veya ağ adresi

8050 portu doluysa:

```powershell
conda run -n tep tep ui --port 8051
```

Yalnızca bu bilgisayardan erişim için varsayılan `127.0.0.1` en güvenli seçimdir. Yerel ağdan erişim özellikle isteniyorsa:

```powershell
conda run -n tep tep ui --host 0.0.0.0 --port 8050
```

`0.0.0.0` kullanımı uygulamayı yerel ağa açabilir; güvenlik duvarı ve ağ koşulları dikkate alınmalıdır.

### 2.3 Sunucuyu kapatma

Sunucu ön planda çalışıyorsa terminalde `Ctrl+C` kullanın.

8050 portunu kullanan işlemi bulmak için:

```powershell
Get-NetTCPConnection -LocalPort 8050 -State Listen | Select-Object OwningProcess
```

Sonra yalnızca doğru işlem olduğundan emin olarak:

```powershell
Stop-Process -Id <PID>
```

## 3. TEP Studio neyi simüle eder?

Tennessee Eastman Process (TEP), plant-wide kontrol, arıza teşhisi, süreç izleme, optimizasyon ve pekiştirmeli öğrenme için kullanılan dinamik bir kimya tesisi benchmark'ıdır.

Modelde beş ana proses ünitesi bulunur:

1. **Reaktör:** Ekzotermik gaz fazı reaksiyonlarının gerçekleştiği ünite.
2. **Kondenser:** Reaktör çıkışını soğutarak ağır ürünlerin yoğuşmasını sağlar.
3. **Vapor/liquid separator:** Yoğuşan sıvıyı gaz fazından ayırır.
4. **Recycle compressor:** Gaz fazını tekrar reaktör basıncına yükseltir.
5. **Stripper:** Sıvı üründeki hafif bileşenleri uzaklaştırır.

Kimyasal bileşenler ticari bilgiyi gizlemek amacıyla `A`–`H` olarak adlandırılmıştır. `G` ve `H` ürünler, `F` yan ürün, `B` ise inert bileşendir.

Modelin boyutları:

- 50 dahili durum,
- 41 yayımlanmış ölçüm,
- 12 manipüle edilen değişken (MV),
- 28 standart bozucu/arıza (IDV),
- 6 hazır işletim modu.

TEP açık çevrimde kararsızdır. Kontrol uygulanmadığında proses genellikle reaktör basıncı emniyet sınırına ulaşarak kapanır. Bu davranış bir hata değil, benchmark'ın temel özelliğidir.

## 4. Temel kavramlar

### 4.1 Measurement — ölçüm

Kontrolörün veya bir ML/RL ajanının görebildiği 41 proses sinyalidir. Örnekler:

- reaktör basıncı, seviyesi ve sıcaklığı,
- separator ve stripper seviyeleri,
- besleme ve geri dönüş akışları,
- purge akışı,
- ürün ve besleme bileşimleri,
- kompresör işi,
- soğutma suyu çıkış sıcaklıkları.

Python'da listelemek için:

```python
import tep_studio as tep

for name, unit, description in tep.list_measurements():
    print(name, unit, description)
```

CLI karşılığı:

```powershell
conda run -n tep tep list measurements
```

### 4.2 Manipulated variable — MV

Kontrolörün değiştirebildiği 12 aktüatördür. Değerler yüzde `0..100` aralığındadır:

1. `d_feed_valve`
2. `e_feed_valve`
3. `a_feed_valve`
4. `ac_feed_valve`
5. `compressor_recycle_valve`
6. `purge_valve`
7. `separator_underflow_valve`
8. `stripper_underflow_valve`
9. `stripper_steam_valve`
10. `reactor_cooling_water_valve`
11. `separator_cooling_water_valve`
12. `reactor_agitator_speed`

Listelemek için:

```powershell
conda run -n tep tep list mvs
```

### 4.3 Setpoint — hedef değer

Kapalı çevrim kontrolörün takip ettiği hedeflerdir:

- `reactor_level`
- `reactor_pressure`
- `reactor_temperature`
- `separator_level`
- `stripper_level`
- `pct_g`
- `production_rate`
- `ya`
- `yac`

Listelemek için:

```powershell
conda run -n tep tep list setpoints
```

### 4.4 IDV — disturbance/fault

IDV'ler modelin standart bozucu ve arıza girişleridir. Örnekler:

- `idv_01`: Stream 4 A/C oranı değişimi,
- `idv_03`: D besleme sıcaklığı değişimi,
- `idv_06`: A besleme kaybı,
- `idv_13`: Reaksiyon kinetiği sürüklenmesi,
- `idv_14`: Reaktör soğutma suyu valfi stiction,
- `idv_15`: Separator soğutma suyu valfi stiction,
- `idv_19`: Valf stiction grubu.

Tam liste:

```powershell
conda run -n tep tep list disturbances
```

### 4.5 Open loop ve closed loop

- **Closed loop:** PI kontrolör ölçümleri okur ve valfleri otomatik hareket ettirir. Kullanıcı setpoint belirler.
- **Open loop:** Kullanıcı 12 valfin konumunu doğrudan belirler. Kontrolör yoktur.

İlk çalışmalar ve güvenilir veri üretimi için kapalı çevrim önerilir. Açık çevrim deneyleri kısa tutulmalıdır.

### 4.6 Terminated ve truncated

- `terminated=True`: Prosesin kendi dinamiği nedeniyle bir shutdown gerçekleşmiştir.
- `truncated=True`: Shutdown olmadan seçilen simülasyon ufkuna ulaşılmıştır.
- `stabilized=True`: Ufka ulaşılmış ve proses kapanmamıştır.

## 5. Altı işletim modu

| Mod | Ürün G:H | Üretim hedefi | Başlangıç |
|---|---:|---|---|
| Mode 1 | 50/50 | Base | Temel durum |
| Mode 2 | 10/90 | Base | Temel durumdan hedefe sürülür |
| Mode 3 | 90/10 | Base | Hazır 90/10 durum vektörü |
| Mode 4 | 50/50 | Max | Temel durumdan hedefe sürülür |
| Mode 5 | 10/90 | Max | Temel durumdan hedefe sürülür |
| Mode 6 | 90/10 | Max | Hazır 90/10 durum vektörü |

Mode 3 ve 6'nın özel başlangıç durumu kullanmasının nedeni, temel durumdan 90/10 bileşimine geçişin yüksek basınç shutdown'ına yol açabilmesidir. “Max” modları temel üretimin yaklaşık %10 üzerine çıkmayı dener; bir besleme valfinin doyması gerçek üretimi sınırlayabilir.

## 6. Kontrol mimarisi

Yerleşik kontrolör, Ricker tarzı merkezi olmayan çok döngülü PI yapısıdır. Tek bir merkezi optimizatör yerine birlikte çalışan çok sayıda SISO PI döngüsü kullanılır.

Başlıca ilişkiler:

- Üretim hızı, genel üretim indeksi üzerinden besleme ve çekiş akışlarını birlikte ölçekler.
- Reaktör sıcaklığı, reaktör soğutma suyu valfiyle kontrol edilir.
- Reaktör basıncı purge oranıyla düzenlenir.
- Separator ve stripper seviyeleri underflow oranlarıyla düzenlenir.
- Reaktör seviyesi, separator sıcaklık hedefine kaskat bağlıdır.
- `%G` hedefi D ve E besleme oranlarını etkiler.
- `yA` ve `yAC` yavaş bileşim trimleridir.

Koruma fonksiyonları:

- Yüksek reaktör basıncında üretim indeksi azaltılır.
- Yüksek reaktör seviyesinde recycle valfine müdahale edilir.
- Üretim ve ürün bileşimi hedefleri ani sıçrama yerine rampalanır.

Web arayüzündeki `overrides` seçeneği bu koruma müdahalelerini etkinleştirir. Normal deneylerde açık bırakılması önerilir.

## 7. Web arayüzü

Ana sekmeler:

1. `Simulate`
2. `Dataset`
3. `Compare`
4. `Metrics / Record`
5. `Assistant`

Başlıktaki `About the TEP` sayfasında proses, kimya, değişkenler ve kontrol stratejisiyle ilgili ek açıklamalar bulunur.

### 7.1 Simulate

Sol panel koşuyu yapılandırır, sağ panel sonuçları gösterir.

#### Run configuration

- **Operating mode:** Mode 1–6.
- **Loop:** Closed veya open loop.
- **Horizon (h):** Simülasyon süresi.
- **Fidelity:**
  - `Explore (Δt=0.01 h)`: hızlı deneme,
  - `Fidelity (Δt=0.0005 h)`: daha yüksek zaman çözünürlüğü.
- **Seed:** Ölçüm gürültüsünü tekrar üretilebilir yapar.
- **Controller flags:** composition, overrides ve `%G feedback`.

`0.01 h = 36 s`, `0.0005 h = 1.8 s` simülasyon zamanıdır. Bunlar gerçek zamanlı bekleme süreleri değildir.

Kapalı çevrim seçildiğinde setpoint alanları, açık çevrim seçildiğinde doğrudan MV sürgüleri görünür.

#### Disturbances

- Bir veya daha fazla IDV seçilebilir.
- `Activation time (h)` bozucunun başlangıç zamanıdır.
- Her IDV için beliren magnitude alanı şiddeti belirler.
- Çoğu IDV latched davranır; etkinleştirildikten sonra koşu boyunca aktif kalır.

#### Advanced: Initial state

Varsayılan mod durumunun yerine 50 elemanlı özel bir başlangıç vektörü kullanılabilir. JSON listesi veya virgül/boşlukla ayrılmış sayılar kabul edilir.

#### Advanced: Controller tuning

PI kazançları, oranlar, override limitleri ve setpoint rampaları düzenlenebilir. `Reset to defaults` yerleşik değerlere döner.

#### Advanced: Solver

- `RK4`: Hızlı ve varsayılan sabit adımlı çözücü.
- `Euler`: Basit sabit adımlı yöntem; hassasiyet açısından dikkatli kullanılmalıdır.
- `RK45`, `RK23`: SciPy tabanlı adaptif çözücüler.
- `fixed_step`: RK4/Euler iç entegrasyon adımı.
- `rtol`, `atol`: Yalnızca adaptif çözücülerde etkilidir.
- `record_every`: Kayıt seyrekleştirmesi; `0` otomatik seçimdir.

#### Grafikler

Üst grafik ölçümleri, alt grafik MV hareketlerini gösterir. `setpoints` ve `limits` katmanları açılıp kapatılabilir.

Özellikle şunları takip edin:

- Reaktör basıncı; shutdown sınırı yaklaşık 3000 kPa gauge,
- Reaktör/separator/stripper seviyeleri,
- Reaktör sıcaklığı,
- Üretim akışı,
- Valflerin 0 veya 100'de doyması.

#### Scenario kaydetme

`Save scenario`, tüm koşu ayarlarını JSON olarak indirir. `Load scenario`, JSON'u doğrulayıp arayüze geri yükler. Bilinmeyen IDV/MV/setpoint adları ve sınır dışı değerler doğrulamada reddedilir.

### 7.2 Dataset

#### Export

Oturumda oluşturulan koşular seçilerek CSV, Parquet veya JSON veri setine dönüştürülebilir. Bir ölçüm, indirme öncesinde koşular arasında üst üste çizilebilir.

#### System ID (excitation)

APC/MPC veya başka bir dinamik model için tasarlanmış uyarım üretir:

- PRBS,
- GBN,
- APRBS,
- multisine,
- chirp.

Kapalı çevrim uyarımı setpoint'lere, açık çevrim uyarımı doğrudan MV'lere uygulanır. Açık çevrim uyarımları kısa tutulmalıdır.

Başlıca parametreler:

- hedef sinyaller,
- güvenli aralığın kesri olarak amplitude,
- clock/hold süresi,
- frekans bandı,
- horizon,
- seed.

#### Batch sweep

Birden fazla seed ve isteğe bağlı parametre taraması üretir. Taranabilen alanlar:

- setpoint alanları,
- horizon,
- control interval,
- fixed step.

Örnek:

```text
Seeds: 1, 2, 3
Sweep parameter: setpoints.production_rate
Sweep values: 22, 28, 32
```

Bu örnek 3 × 3 = 9 koşu oluşturur. Birleşik veri seti ve metrik CSV'si indirilebilir.

### 7.3 Compare

Oturumdaki bütün koşuların özet tablosunu ve üst üste çizimlerini sunar. Tablo; shutdown durumu, son zaman, tepe reaktör basıncı ve hata metriklerini içerir.

İyi bir kullanım örneği:

1. Nominal Mode 1 koşusu oluşturun.
2. Aynı ayarlarla IDV 13 koşusu oluşturun.
3. `Compare` içinde reaktör basıncı, sıcaklığı ve üretimi üst üste çizin.

`Clear all runs`, yalnızca web oturumundaki koşu deposunu temizler.

### 7.4 Metrics / Record

Koşu seçildiğinde aşağıdaki göstergeler incelenebilir:

- tepe reaktör basıncı,
- minimum/maksimum seviyeler,
- IAE — integral absolute error,
- ISE — integral squared error,
- sınır ihlali adımı sayısı,
- shutdown'a kadar geçen süre,
- toplam işletme maliyeti göstergesi.

IAE ve ISE için genel olarak daha düşük değer daha iyi hedef takibi demektir. Ancak performans, güvenlik, üretim ve maliyet birlikte değerlendirilmelidir.

`Experiment record`, koşunun setpoint, kontrol ayarları, metrikleri ve tekrar üretim bilgilerini içeren JSON kaydıdır.

### 7.5 Assistant

Doğal dilde simülasyon istemek için deneysel bir arayüzdür. Örneğin:

```text
Run Mode 1 closed-loop with IDV13 starting at 2 h for 10 hours and report peak reactor pressure.
```

Bu bölüm:

- deneysel ve tam güvenilir değildir,
- Anthropic API anahtarı ister,
- `agent` extra paketinin kurulmasını gerektirir,
- ana simülasyon için zorunlu değildir.

Mevcut kurulumda `ui` extra kuruludur; Assistant/MCP kullanılacaksa ayrıca:

```powershell
conda run -n tep python -m pip install "tep-studio[agent]==0.2.0"
```

API anahtarları kod veya rehber dosyasına yazılmamalıdır.

## 8. CLI kullanımı

Genel yardım:

```powershell
conda run -n tep tep --help
```

### 8.1 Tek koşu

Nominal kapalı çevrim Mode 1:

```powershell
conda run -n tep tep run --mode mode1 --loop closed --horizon 12 --overrides
```

IDV 13, ikinci saatte:

```powershell
conda run -n tep tep run --mode mode1 --horizon 12 --idv idv_13@2.0 --overrides
```

Birden fazla IDV:

```powershell
conda run -n tep tep run --horizon 12 --idv idv_03@1.0 --idv idv_13@4.0 --overrides
```

Setpoint override:

```powershell
conda run -n tep tep run --horizon 12 --setpoint production_rate=25 --setpoint pct_g=55 --overrides
```

Sonucu Parquet dosyasına yazma:

```powershell
conda run -n tep tep run --mode mode1 --horizon 12 --overrides --out mode1.parquet
```

> Önemli: Web arayüzünde safety override'lar varsayılan olarak açıktır. CLI'da `--overrides` bayrağını açıkça vermek gerekir.

### 8.2 Açık çevrim koşusu

```powershell
conda run -n tep tep run --loop open --horizon 1 --mv purge_valve=45 --mv reactor_cooling_water_valve=60 --out open_loop.csv
```

Belirtilmeyen MV'ler nominal başlangıç değerinde kalır. Açık çevrim kararsız olduğundan kısa ufukla başlamak önerilir.

### 8.3 Çok seed'li veri seti

```powershell
conda run -n tep tep dataset --mode mode1 --horizon 12 --seeds 1,2,3,4 --overrides --out dataset.parquet
```

İşlemci sayısını sınırlandırmak için:

```powershell
conda run -n tep tep dataset --horizon 12 --seeds 1,2,3,4 --overrides --out dataset.parquet --workers 2
```

### 8.4 FDD benchmark üretimi

Bütün 28 arıza için:

```powershell
conda run -n tep tep benchmark --out fdd_benchmark.parquet
```

Seçili arızalar için:

```powershell
conda run -n tep tep benchmark --out fdd_subset.parquet --faults idv_01,idv_06,idv_13 --runs 5 --onset 8 --horizon 48 --sampling-min 3
```

### 8.5 Offline RL veri dışa aktarımı

```powershell
conda run -n tep tep rl-export --mode mode1 --horizon 24 --idv idv_13@4 --overrides --out transitions.parquet
```

Desteklenen çıktı uzantıları `.npz` ve `.parquet`tir.

## 9. Python API

### 9.1 Hızlı kapalı çevrim testi

```python
import tep_studio as tep

summary = tep.quickstart(horizon=8.0, control_interval=0.01)
print(summary)
```

### 9.2 Kapalı çevrim simülasyon

```python
from tep_studio import ClosedLoopSimulation

simulation = ClosedLoopSimulation(
    horizon=24.0,
    control_interval=0.01,
)

result = simulation.run(seed=1, mode="mode1", record_every=5)

print("stabilized:", result.stabilized)
print("terminated:", result.terminated)
print("final time:", result.final_time)
print("peak pressure:", result.peak["reactor_pressure_max"])
print("metrics:", result.metrics)
```

`record_every`, döndürülen zaman serisini seyrekleştirir; tepe değerleri ve metrikler yine tam çözünürlükte hesaplanır.

### 9.3 Zamanlanmış disturbance

```python
import numpy as np
from tep_studio import ClosedLoopSimulation

def disturbances(time_h: float) -> np.ndarray:
    idv = np.zeros(28)
    if time_h >= 2.0:
        idv[12] = 1.0  # idv_13: Python dizininde 12
    return idv

result = ClosedLoopSimulation(
    horizon=12.0,
    control_interval=0.01,
).run(
    seed=1,
    mode="mode1",
    disturbance_schedule=disturbances,
    record_every=5,
)

print(result.stabilized, result.peak)
```

IDV adları 1'den, NumPy indisleri 0'dan başladığı için `idv_13` dizide `12` numaralı indiste bulunur.

### 9.4 Düşük seviye proses API'si

```python
import numpy as np
from tep_studio import TennesseeEastmanProcess

plant = TennesseeEastmanProcess(
    solver_method="RK4",
    fixed_step=0.0005,
)

measurements, info = plant.reset(mode="mode1", seed=1)

# Başlangıç durumundaki nominal 12 MV değerini kullan.
action = plant.state[38:50].copy()

result = plant.advance(
    action,
    control_interval=0.01,
    disturbances=np.zeros(28),
)

print(result.time)
print(result.measurements)
print(result.constraint_margins)
print(result.shutdown_status)
print(result.objective_terms)
```

`advance()` tarafından döndürülen `AdvanceResult` şunları içerir:

- 50 elemanlı state,
- 41 measurement,
- requested ve implemented action,
- disturbance vektörü,
- constraint margin'leri,
- olaylar ve shutdown durumu,
- solver istatistikleri,
- objective/cost terimleri.

MV değerleri otomatik olarak `0..100` aralığına kırpılır.

### 9.5 Yüksek seviye Scenario API'si

Bu API, web arayüzünün Dash bağımsız backend'idir:

```python
from tep_studio.analysis import (
    DisturbanceActivation,
    ScenarioConfig,
    run_scenario,
)

config = ScenarioConfig(
    name="mode1_idv13",
    mode="mode1",
    loop_type="closed",
    horizon=12.0,
    control_interval=0.01,
    seed=1,
    enable_composition=True,
    enable_overrides=True,
    disturbances=(
        DisturbanceActivation(
            idv="idv_13",
            magnitude=1.0,
            start_time=2.0,
        ),
    ),
)

run = run_scenario(config)
frame = run.to_frame()

print(run.run_id)
print(run.summary())
print(frame.head())
```

### 9.6 Step test

Kapalı çevrim setpoint adımı:

```python
from tep_studio.analysis import (
    ScenarioConfig,
    StepTestSpec,
    run_setpoint_step_test,
)

config = ScenarioConfig(
    name="production_step",
    loop_type="closed",
    horizon=8.0,
    control_interval=0.01,
    enable_overrides=True,
)

step = StepTestSpec(
    kind="setpoint",
    target="production_rate",
    baseline=22.89,
    step_value=25.0,
    step_time=2.0,
)

run = run_setpoint_step_test(config, step)
run.to_frame().to_parquet("production_step.parquet", index=False)
```

Açık çevrim MV adımı için `kind="mv"`, örneğin `target="d_feed_valve"`, ve `run_mv_step_test()` kullanılır.

### 9.7 Batch API

```python
from tep_studio.analysis import BatchSpec, ScenarioConfig, run_batch

base = ScenarioConfig(
    name="production_sweep",
    mode="mode1",
    horizon=8.0,
    control_interval=0.01,
    enable_overrides=True,
)

spec = BatchSpec(
    base=base,
    seeds=(1, 2, 3),
    param_grid={
        "setpoints.production_rate": (22.0, 26.0, 30.0),
    },
    label="prod",
)

batch, runs = run_batch(spec, max_workers=2)

print(batch.batch_id)
for run in runs:
    print(run.summary())
```

### 9.8 TrajectoryDataset

```python
from tep_studio import ClosedLoopSimulation, TrajectoryDataset

result = ClosedLoopSimulation(
    horizon=4.0,
    control_interval=0.01,
).run(record_every=5)

dataset = TrajectoryDataset.from_results(
    result.results,
    run_id="run_001",
    scenario_id="nominal_mode1",
    labels={"fault": "none"},
)

frame = dataset.to_pandas()
measurements, columns = dataset.to_numpy("measurement")

dataset.to_csv("trajectory.csv")
dataset.to_parquet("trajectory.parquet")
```

Tidy veri setindeki sütun grupları:

- `measurement.*`
- `state.*`
- `requested_action.*`
- `implemented_action.*`
- `disturbance.*`
- `objective.*`
- run/scenario/time/lifecycle alanları.

## 10. Gymnasium ve RL

Kütüphane import edildiğinde `TennesseeEastman-v0` ortamı kaydedilir.

### 10.1 Doğrudan valf kontrolü

```python
import gymnasium as gym
import tep_studio  # ortam kaydını yapar

env = gym.make(
    "TennesseeEastman-v0",
    horizon=4.0,
    control_interval=0.01,
    mode="mode1",
)

observation, info = env.reset(seed=1)

terminated = False
truncated = False

while not (terminated or truncated):
    action = env.action_space.sample()
    observation, reward, terminated, truncated, info = env.step(action)

env.close()
```

- Observation: 41 ölçüm.
- Action: 12 adet `0..100` MV.
- Varsayılan reward: adım içindeki üretim maliyetinin negatifi.
- Shutdown: `terminated=True`.
- Horizon: `truncated=True`.

Rastgele doğrudan valf hareketleri tesisi kolayca kapatabilir; bu beklenen davranıştır.

### 10.2 Setpoint düzeyinde RL

Daha korumalı bir ajan, doğrudan valf yerine yerleşik PI kontrolörün setpoint'lerini seçebilir:

```python
import gymnasium as gym
import tep_studio

env = gym.make(
    "TennesseeEastman-v0",
    action_level="setpoint",
    setpoint_fields=("production_rate", "pct_g"),
    horizon=12.0,
    control_interval=0.01,
)

obs, info = env.reset(seed=1)
action = env.action_space.sample()
obs, reward, terminated, truncated, info = env.step(action)
```

Desteklenen setpoint action alanları üretim, `%G`, reaktör basıncı/seviyesi/sıcaklığı, separator seviyesi ve stripper seviyesidir.

Özel reward fonksiyonu da `GymTEPEnv(reward_fn=...)` ile verilebilir. Reward fonksiyonu tam `AdvanceResult` nesnesini alır.

## 11. Optimizasyon ve lineerleştirme

`OptimizationAdapter` deterministik rollout, sonlu fark gradyanı ve yerel lineer model üretir.

```python
from tep_studio import OptimizationAdapter, TennesseeEastmanProcess

plant = TennesseeEastmanProcess(rtol=1e-10, atol=1e-12)
plant.reset(mode="mode1")

u0 = plant.state[38:50].copy()
adapter = OptimizationAdapter(plant)

A, B = adapter.linearize(
    plant.state,
    u0,
    control_interval=0.001,
)

print(A.shape, B.shape)
```

Bu alan daha ileri düzeydir. Lineerleştirme sonucu seçilen çalışma noktası, zaman adımı ve sayısal toleranslara bağlıdır.

## 12. MCP sunucusu

`agent` extra kurulduktan sonra TEP Studio bir MCP sunucusu olarak çalıştırılabilir:

```powershell
conda run -n tep tep-mcp
```

Sunucu stdio transport kullanır ve şu araçları sağlar:

- `describe_plant`
- `run_scenario`
- `get_run`
- `get_run_series`
- `list_runs`
- `compare_runs`

Genel MCP istemci yapılandırması:

```json
{
  "mcpServers": {
    "tep-studio": {
      "command": "C:\\Users\\ardao\\anaconda3\\envs\\tep\\Scripts\\tep-mcp.exe"
    }
  }
}
```

Her istemcinin yapılandırma dosyası ve formatı farklı olabilir. Windows'ta tam executable yolu kullanmak, conda aktivasyonuna bağımlılığı azaltır.

## 13. Sayısal yöntem ve performans

Varsayılan çözücü sabit adımlı RK4'tür:

- `control_interval`: Kontrolörün veya dış ajanın ne sıklıkta aksiyon ürettiği.
- `fixed_step`: Her kontrol aralığı içindeki ODE entegrasyon alt adımı.
- `record_every`: Kaç kontrol adımında bir zaman serisi satırı saklandığı.

Örneğin:

```text
control_interval = 0.01 h
fixed_step       = 0.0005 h
```

Her kontrol aralığında 20 RK4 alt adımı anlamına gelir.

Öneriler:

- İlk keşifte UI `Explore` veya `control_interval=0.01` kullanın.
- Hassas doğrulamada daha küçük kontrol aralığı kullanın.
- RK4 kararsızlaşırsa `fixed_step` değerini küçültün veya RK45 deneyin.
- Batch üretiminde önce kısa ve az seed'li bir pilot koşu yapın.
- Çok büyük veri setlerinde CSV yerine Parquet tercih edin.

## 14. Tekrar üretilebilir deney kontrol listesi

Bir deney kaydında en az şunları saklayın:

1. TEP Studio sürümü (`tep version`).
2. Mode ve başlangıç durumu.
3. Open/closed loop seçimi.
4. Horizon ve control interval.
5. Solver, fixed step, rtol ve atol.
6. Seed.
7. IDV adı, magnitude ve başlangıç zamanı.
8. Setpoint veya manuel MV override'ları.
9. Controller flags ve tuning değişiklikleri.
10. Shutdown durumu ve metrikler.

Web arayüzündeki `Save scenario` ile `Download record JSON` bu kayıtların büyük bölümünü otomatik sağlar.

## 15. Önerilen öğrenme sırası

### Deney 1 — nominal çalışma

- Mode 1
- Closed loop
- Horizon 12 h
- Explore
- Disturbance yok
- Composition ve overrides açık

Amaç: Normal basınç, sıcaklık, seviye ve valf davranışını tanımak.

### Deney 2 — kontrolsüz tesis

- Mode 1
- Open loop
- Horizon 2 h
- Nominal MV değerleri

Amaç: Açık çevrim kararsızlığı ve shutdown mekanizmasını görmek.

### Deney 3 — kinetik sürüklenmesi

- Deney 1'in aynısı
- IDV 13
- Activation time 2 h
- Magnitude 1.0

Amaç: Kontrolörün yavaş proses bozulmasına verdiği tepkiyi incelemek.

### Deney 4 — nominal ve arızalı koşuyu karşılaştırma

`Compare` sekmesinde reaktör basıncı, sıcaklığı, `%G` ve üretim akışını üst üste çizin. `Metrics / Record` üzerinden IAE, ISE, tepe basınç ve maliyeti karşılaştırın.

### Deney 5 — veri seti

Seed `1,2,3,4,5` ile nominal ve birkaç IDV koşusu üretin. Parquet indirin ve pandas ile inceleyin.

## 16. Sorun giderme

### `ModuleNotFoundError: tep_studio`

Yanlış Python çalışıyordur. Doğru executable:

```powershell
conda run -n tep python -c "import sys; print(sys.executable)"
```

Beklenen yol:

```text
C:\Users\ardao\anaconda3\envs\tep\python.exe
```

### `tep` komutu bulunamıyor

Ortamı aktive etmek yerine:

```powershell
conda run -n tep tep --help
```

kullanın.

### Web sayfası açılmıyor

Önce sunucunun çalıştığı terminali kontrol edin. Sonra:

```powershell
Test-NetConnection 127.0.0.1 -Port 8050
```

Port doluysa `--port 8051` kullanın.

### UI bağımlılığı eksik

```powershell
conda run -n tep python -m pip install "tep-studio[ui]==0.2.0"
```

### Assistant veya MCP açılmıyor

```powershell
conda run -n tep python -m pip install "tep-studio[agent]==0.2.0"
```

Assistant ayrıca geçerli bir Anthropic API anahtarı ister.

### Native modül yüklenmiyor

Kurulumdaki native dosya şu yapıda olmalıdır:

```text
...\envs\tep\Lib\site-packages\tep_studio\simulation\_tep_native.pyd
```

Kontrol:

```powershell
conda run -n tep python -c "from tep_studio.simulation import native; print('native OK')"
```

Gerekirse paketi aynı ortamda yeniden kurun. TEP Studio 0.2.0 kaynak dağıtımından kurulduğu için Visual Studio C++ Build Tools gerekir; bu bilgisayarda kurulu ve ilk derleme başarıyla tamamlanmıştır.

### Simülasyon diverge ediyor

- MV ve başlangıç durumunu kontrol edin.
- `fixed_step` değerini küçültün.
- RK45 veya RK23 deneyin.
- Open loop koşusunu kısaltın.
- `overrides` seçeneğini etkinleştirin.

### Simülasyon çok yavaş

- UI'da `Explore` kullanın.
- CLI'da `control_interval=0.01` kullanın.
- Gereksiz uzun horizon'dan kaçının.
- Batch worker sayısını kontrollü artırın.
- CSV yerine Parquet kullanın.

### Bağımlılık kontrolü

```powershell
conda run -n tep python -m pip check
conda run -n tep python -m pip show tep-studio
```

## 17. Güncelleme ve ortamı dışa aktarma

Yeni sürüme geçmeden önce sürüm notlarını kontrol edin. Mevcut çalışmaları tekrar üretmek için `0.2.0` pinini korumak daha güvenlidir.

Ortam kaydı:

```powershell
conda env export -n tep > tep-environment.yml
```

Kurulu Python paketleri:

```powershell
conda run -n tep python -m pip freeze > tep-requirements.txt
```

Başka makinede temel yeniden kurulum:

```powershell
conda create -n tep python=3.12 pip -y
conda run -n tep python -m pip install "tep-studio[ui]==0.2.0"
```

TEP Studio 0.2.0 native uzantı içerdiği için hedef işletim sisteminde uygun C derleyicisi gerekebilir.

## 18. Kaynaklar

- Proje sayfası: <https://khalidlabs.com/tep-studio/>
- GitHub: <https://github.com/khalidlabs/tep-studio>
- PyPI: <https://pypi.org/project/tep-studio/>

---

Bu rehber, bu bilgisayarda doğrulanan TEP Studio **0.2.0** kurulumu esas alınarak hazırlanmıştır.

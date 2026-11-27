# Hafta 02 – ADC ile Potansiyometre Okuma ve TIM1 PWM ile LED Parlaklık Kontrolü

**Tarih:** 25.11.2026  
**Mikrodenetleyici / Kart:** STM32F103RBT6 (NUCLEO-F103RB)  
**Geliştirme Ortamı:** STM32CubeMX, STM32CubeIDE, Proteus 8.13  

---

## 1. Çalışma Amacı
Bu çalışmada `STM32F103RBT6` mikrodenetleyicisinin 12-bit çözünürlüklü `ADC1` birimi üzerinden analog potansiyometre gerilimi okunmuş, fiziksel gerilim değeri hesaplanmış ve `TIM1_CH1` PWM çıkışı kullanılarak harici LED parlaklığının %0 – %100 aralığında kontrol edilmesi hem fiziksel devre hem de Proteus simülasyon ortamında doğrulanmıştır.

---

## 2. Klasör İçeriği
* `LED/` — STM32CubeIDE proje dosyaları (`Core/`, `Drivers/`, `Debug/`, `LED.ioc`, `STM32F103RBTX_FLASH.ld`)
* `LED.pdsprj` — Proteus 8.13 simülasyon şeması (`STM32F103R6` modeli ile)
* `LED.hex` — Proteus simülasyonu için 10 KB RAM sınırına göre derlenmiş güncel ikili dosya
* `README.md` — Haftalık çalışma dokümantasyonu

---

## 3. Donanım ve Pin Bağlantıları

| STM32 Pini | NUCLEO Etiketi | Çevre Birimi | Devre Bağlantısı | Açıklama |
| :--- | :--- | :--- | :--- | :--- |
| `3V3` | `3V3` | Güç | Potansiyometre 1. dış bacak | +3.3 V besleme ve referans hattı |
| `GND` | `GND` | Toprak | Potansiyometre 3. bacak & LED katodu | Ortak toprak referansı (0 V) |
| `PA0` | `A0` | `ADC1_IN0` | Potansiyometre orta bacak | 0 – 3.3 V analog gerilim girişi |
| `PA8` | `D7` | `TIM1_CH1` | 330 Ω direnç üzerinden LED anodu | ~122,07 Hz PWM çıkışı (`ARR = 65535`) |

---

## 4. Saat ve Zamanlayıcı Parametreleri
* **SYSCLK / APB2 Timer Clock:** 8 MHz
* **ADC1 Clock:** 4 MHz (APB2 / 2)
* **TIM1 Prescaler (PSC):** `0`
* **TIM1 Counter Period (ARR):** `65535`
* **PWM Frekansı:** $f_{PWM} = \frac{8.000.000}{(0 + 1) \times (65535 + 1)} \approx 122,07\text{ Hz}$
* **Gerilim Dönüşümü:** $V_{in} = \frac{\text{adc\_val}}{4095} \times 3.3\text{ V}$
* **CCR Ölçekleme:** `pwm_ccr = (uint16_t)((adc_val * (uint32_t)htim1.Init.Period) / 4095U)`

---

## 5. Deneysel Ölçüm Sonuçları (Live Expressions)

| Potansiyometre Konumu | Okunan `adc_val` | Hesaplanan Gerilim (`voltage`) | Ölçülen `pwm_ccr` | Duty Cycle (%) |
| :--- | :---: | :---: | :---: | :---: |
| Orta Konum (~%49,6) | 2031 | 1.637 V | 32503 | %49,60 |
| Maksimum Konum (%100) | 4095 | 3.300 V | 65535 | %100,00 |

---

## 6. Proteus Simülasyon Notu (10 KB SRAM Düzenlemesi)
Proteus kütüphanesinde `STM32F103RBT6` (20 KB SRAM) modeli yer almadığı için eşdeğer pin dizilimine sahip `STM32F103R6` (10 KB SRAM) modeli kullanılmıştır. İlk denemede yığın işaretçisinin `0x20004FDC` adresine erişmeye çalışmasından kaynaklanan `[INTERRUPT] Stack error while saving exception context` hatasını gidermek amacıyla `STM32F103RBTX_FLASH.ld` dosyasında `RAM` uzunluğu `10K` olarak düzenlenip yeniden derleme alınmıştır.
ssc install xtabond2
encode Negara, gen(Nat)
xtset Nat Tahun
#Gunakan GMM Arellano-Bond dynamics
xtabond INFit FTA OPENIM GDP RER MSG FTAXOPEN, lags(1) artests(2)
#Uji validitas
estat sargan
#Uji konsistensi
estat abond
#Jika tidak berjalan
xtabond INFit FTA OPENIM GDP RER MSG FTAXOPEN, lags(1) vce(robust) artests(2)
estat abond
#Run ulang dan simpan hasil estimasi
xtabond INFit FTA OPENIM GDP RER MSG FTAXOPEN, lags(1) vce(robust) artests(2)
estimates store fdgmm
#Gunakan estimasi kedua dengan fixed effect dan simpan hasil estimasi
xtreg INFit L1.INFit FTA OPENIM GDP RER MSG FTAXOPEN, fe
estimates store fem
#Gunakan estimasi ketiga dengan regression dan simpan hasil estimasi
regress INFit L1.INFit FTA OPENIM GDP RER MSG FTAXOPEN 
estimates store pls
#Bandingkan ketiga hasil estimasi
estimates table fdgmm fem pls, star stats(N)
#Agar tidak bias, nilai estimasi fdgmm harus ada di antara fem dan pls
#Lakukan perhitungan kedua dengan System GMM
xtdpdsys INFit FTA OPENIM GDP RER MSG FTAXOPEN, lags(1) artests(2)
#Uji validitas
estat sargan
#Lakukan penyesuaian
xtdpdsys INFit FTA OPENIM GDP RER MSG FTAXOPEN, lags(1) vce(robust) artests(2)
#Uji konsistensi
estat abond
#Run ulang dan simpan
xtdpdsys INFit FTA OPENIM GDP RER MSG FTAXOPEN, lags(1) vce(robust) artests(2)
estimates store sysgmm
#Bandingkan empat hasil estimasi untuk mengecek ketidakbiasan
estimates table fdgmm sysgmm fem pls, star stats(N) 
#Akan dicheck seberapa cepat dependent variabel berubah
#Gunakan angka coeff dari lag variable dependent pada sysgmm
display -log(.53749986)
#Didapat hasil .62082678 yang bermakna dependent variable akan berubah secepat 62,08% per tahunnya.
#Nilai coeff dari masing-masing variabel adalah besar pengaruh jangka pendek variabel bebas pada variabel terikat
#Untuk menghitung pengaruh jangka panjangnya, bisa digunakan perbandingan antara ceff jangka pendek/1-lagindependent, gunakan fungsi
#Run ulang dulu yang sysgmm lalu hitung pengaruh jangka panjangnya.
#Sebagai contoh ambil GDP
nlcom _b[RER]/(1-_b[L1.INFit])
#Nilai coeff adalah nilai pengaruh jangka panjang dari variabel GDP
#Lanjutkan menghitung pengaruh jangka panjang dari seluruh variabel bebas, namun FTA dan FTA tidak diikutertakan karena masalah kolinieritas.
nlcom (_b[FTA]/(1-_b[L1.INFit]))(_b[OPENIM]/(1-_b[L1.INFit]))(_b[GDP]/(1-_b[L1.INFit]))(_b[RER]/(1-_b[L1.INFit]))(_b[MSG]/(1-_b[L1.INFit]))(_b[FTAXOPEN]/(1-_b[L1.INFit]))
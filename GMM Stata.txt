ssc install xtabond2
encode Negara, gen(Nat)
xtset Nat Tahun
gen lnINFit=log(INFit)
gen lnOPENIM=log(OPENIM)
gen lnGDP=log(GDP)
gen lnRER=log(RER)
gen lnMSG=log(MSG)

#Gunakan GMM Arellano-Bond dynamics
xtabond lnINFit FTA lnOPENIM lnGDP lnRER lnMSG FTAXOPEN, lags(1) artests(2)
#Uji validitas
estat sargan
#Uji konsistensi
estat abond
#Jika tidak berjalan
xtabond lnINFit FTA lnOPENIM lnGDP lnRER lnMSG FTAXOPEN, lags(1) vce(robust) artests(2)
estat abond
#Run ulang dan simpan hasil estimasi
xtabond lnINFit FTA lnOPENIM lnGDP lnRER lnMSG FTAXOPEN, lags(1) vce(robust) artests(2)
estimates store fdgmm
#Gunakan estimasi kedua dengan fixed effect dan simpan hasil estimasi
xtreg lnINFit L1.lnINFit FTA lnOPENIM lnGDP lnRER lnMSG FTAXOPEN, fe
estimates store fem
#Gunakan estimasi ketiga dengan regression dan simpan hasil estimasi
regress lnINFit L1.lnINFit FTA lnOPENIM lnGDP lnRER lnMSG FTAXOPEN 
estimates store pls
#Bandingkan ketiga hasil estimasi
estimates table fdgmm fem pls, star stats(N)
#Agar tidak bias, nilai estimasi fdgmm harus ada di antara fem dan pls
#Lakukan perhitungan kedua dengan System GMM
xtdpdsys lnINFit FTA lnOPENIM lnGDP lnRER lnMSG FTAXOPEN, lags(1) artests(2)
#Uji validitas
estat sargan
#Lakukan penyesuaian
xtdpdsys lnINFit FTA lnOPENIM lnGDP lnRER lnMSG FTAXOPEN, lags(1) vce(robust) artests(2)
#Uji konsistensi
estat abond
#Run ulang dan simpan
xtdpdsys lnINFit FTA lnOPENIM lnGDP lnRER lnMSG FTAXOPEN, lags(1) vce(robust) artests(2)
estimates store sysgmm
#Bandingkan empat hasil estimasi untuk mengecek ketidakbiasan
estimates table fdgmm sysgmm fem pls, star stats(N) 
#Akan dicheck seberapa cepat dependent variabel berubah
#Gunakan angka coeff dari lag variable dependent pada sysgmm
display -log(.78400949)
#Didapat hasil .25417666 yang bermakna dependent variable akan berubah secepat 25,42% per tahunnya.
#Nilai coeff dari masing-masing variabel adalah besar pengaruh jangka pendek variabel bebas pada variabel terikat
#Untuk menghitung pengaruh jangka panjangnya, bisa digunakan perbandingan antara ceff jangka pendek/1-lagindependent, gunakan fungsi
#Run ulang dulu yang sysgmm lalu hitung pengaruh jangka panjangnya.
#Sebagai contoh ambil GDP
nlcom _b[lnRER]/(1-_b[L1.lnINFit])
#Nilai coeff adalah nilai pengaruh jangka panjang dari variabel GDP
#Lanjutkan menghitung pengaruh jangka panjang dari seluruh variabel bebas, namun FTA dan FTA tidak diikutertakan karena masalah kolinieritas.
nlcom (_b[FTA]/(1-_b[L1.lnINFit]))(_b[lnOPENIM]/(1-_b[L1.lnINFit]))(_b[lnGDP]/(1-_b[L1.lnINFit]))(_b[lnRER]/(1-_b[L1.lnINFit]))(_b[lnMSG]/(1-_b[L1.lnINFit]))(_b[FTAXOPEN]/(1-_b[L1.lnINFit]))
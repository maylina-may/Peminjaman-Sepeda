# **Peminjaman-Sepeda**

**Deskripsi**


Proyek ini berupa dashboard streamlit yang dikembangkan untuk menganalisis kualitas Peminjaman Sepeda, menggunakan dataset (bike-sharing) yang telah disediakan oleh dicoding pada modul analisis data dengan python.

Setup Environment - Shell/Terminal

mkdir proyek_analisis_data
cd proyek_analisis_data
pipenv install
pipenv shell
pip install -r requirements.txt

Menjalankan Aplikasi Streamlit

streamlit run dashboard.py

Tahapan Analisis Data
1. Memuat Dataset

import pandas as pd

# URL ke file CSV di GitHub
csv_url = "https://raw.githubusercontent.com/maylina-may/Peminjaman-Sepeda/main/hour.csv"

# Membaca dataset dari URL
data_df = pd.read_csv(csv_url)

2. Menampilkan Data Awal

# Menampilkan data
data_df.head()

3. Memeriksa Informasi Dataset

# Melihat informasi tentang dataset
data_df.info()

4. Mengecek Nilai yang Hilang dan Duplikasi

# Mengecek nilai yang hilang
missing_values = data_df.isnull().sum()

# Memeriksa duplikasi data
duplicate_count = data_df.duplicated().sum()

5. Membersihkan Data

# Menghapus semua baris yang memiliki nilai hilang
data_df.dropna(inplace=True)

6. Analisis Pertanyaan Bisnis
Pertanyaan 1: Apa hari dalam seminggu yang paling banyak peminjam sepeda?

# Menambahkan kolom hari dalam seminggu
data_df['dteday'] = pd.to_datetime(data_df['dteday'])
data_df['Day_Of_Week'] = data_df['dteday'].dt.day_name()

# Menghitung jumlah peminjam sepeda berdasarkan hari
day_counts = data_df.groupby('Day_Of_Week')['cnt'].sum().reindex(['Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday', 'Sunday'])

Visualisasi Pertanyaan 1

import matplotlib.pyplot as plt

# Visualisasi jumlah peminjam sepeda per hari dalam seminggu
plt.figure(figsize=(8, 5))
day_counts.plot(kind='bar', color='skyblue')
plt.title('Jumlah Peminjam Sepeda per Hari dalam Seminggu')
plt.xlabel('Hari')
plt.ylabel('Jumlah Peminjam')
plt.xticks(rotation=40)
plt.show()

Pertanyaan 2: Bagaimana tren peminjaman sepeda per bulan?

# Menambahkan kolom bulan
data_df['Month'] = data_df['dteday'].dt.month_name()

# Menghitung jumlah peminjam sepeda berdasarkan bulan
month_counts = data_df.groupby('Month')['cnt'].sum().reindex(['January', 'February', 'March', 'April', 'May', 'June', 'July', 'August', 'September', 'October', 'November', 'December'])

Visualisasi Pertanyaan 2

# Visualisasi jumlah peminjam sepeda per bulan
plt.figure(figsize=(8, 5))
month_counts.plot(kind='bar', color='lightgreen')
plt.title('Jumlah Peminjam Sepeda per Bulan')
plt.xlabel('Bulan')
plt.ylabel('Jumlah Peminjam')
plt.xticks(rotation=40)
plt.show()

Pertanyaan 3: Bagaimana peminjaman sepeda berdasarkan musim?

# Menambahkan kolom musim
def get_season(month):
    if month in [12, 1, 2]:
        return 'Winter'
    elif month in [3, 4, 5]:
        return 'Spring'
    elif month in [6, 7, 8]:
        return 'Summer'
    else:
        return 'Fall'

data_df['Season'] = data_df['dteday'].dt.month.apply(get_season)

# Menghitung jumlah peminjam sepeda berdasarkan musim
season_counts = data_df.groupby('Season')['cnt'].sum()

Visualisasi Pertanyaan 3

# Visualisasi jumlah peminjam sepeda per musim
plt.figure(figsize=(8, 5))
season_counts.plot(kind='bar', color='salmon')
plt.title('Jumlah Peminjam Sepeda per Musim')
plt.xlabel('Musim')
plt.ylabel('Jumlah Peminjam')
plt.xticks(rotation=0)
plt.show()

7. Analisis RFM
Siapa pelanggan terbaik berdasarkan RFM?

# Menghitung RFM
rfm_df = data_df.groupby(by="instant", as_index=False).agg({
    "dteday": "max",  # Mengambil tanggal peminjaman terakhir
    "cnt": "count"    # Menghitung jumlah peminjaman
})

rfm_df.columns = ["instant", "max_borrow_date", "frequency"]

# Menghitung Recency
recent_date = data_df['dteday'].max()
rfm_df["recency"] = rfm_df["max_borrow_date"].apply(lambda x: (recent_date - x).days)

# Menghapus kolom max_borrow_date
rfm_df.drop("max_borrow_date", axis=1, inplace=True)

Visualisasi Pelanggan Terbaik

import seaborn as sns

# Visualisasi pelanggan terbaik berdasarkan RFM
fig, ax = plt.subplots(nrows=1, ncols=3, figsize=(30, 6))

colors = ["#72BCD4"] * 5  # Warna untuk visualisasi

# Visualisasi berdasarkan Recency
sns.barplot(y="recency", x="instant", data=rfm_df.sort_values(by="recency", ascending=True).head(5), hue="instant", palette=colors, dodge=False, ax=ax[0], legend=False)
ax[0].set_ylabel(None)
ax[0].set_xlabel(None)
ax[0].set_title("By Recency (days)", loc="center", fontsize=18)
ax[0].tick_params(axis='x', labelsize=15)

# Visualisasi berdasarkan Frequency
sns.barplot(y="frequency", x="instant", data=rfm_df.sort_values(by="frequency", ascending=False).head(5), hue="instant", palette=colors, dodge=False, ax=ax[1], legend=False)
ax[1].set_ylabel(None)
ax[1].set_xlabel(None)
ax[1].set_title("By Frequency", loc="center", fontsize=18)
ax[1].tick_params(axis='x', labelsize=15)

# Menghitung nilai 'monetary' (misalnya, total cnt untuk setiap instant)
rfm_df['monetary'] = data_df.groupby('instant')['cnt'].sum().values

# Visualisasi berdasarkan Monetary
sns.barplot(y="monetary", x="instant", data=rfm_df.sort_values(by="monetary", ascending=False).head(5), hue="instant", palette=colors, dodge=False, ax=ax[2], legend=False)
ax[2].set_ylabel(None)
ax[2].set_xlabel(None)
ax[2].set_title("By Monetary", loc="center", fontsize=18)
ax[2].tick_params(axis='x', labelsize=15)

plt.suptitle("Best Customers Based on RFM Parameters (instant)", fontsize=20)
plt.show()

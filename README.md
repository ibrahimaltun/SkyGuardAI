# SkyGuardAI: Havacılık Bakım ve Operasyon Asistanı
- Bu projeyle birlikte, hem prediction hem de akıllı döküman analizi (GenAI) yeteneklerini birleştirerek bir "Agentic" yapı kurmak hedeflenmiştir.
- Bu platform;
  - uçak motor verilerini analiz ederek arıza tahmini yapar ve
  - teknik manuel dökümanları kullanarak teknisyenlere tamir adımları önerir.
  

## Data Links:
- Veri indirme linki: https://data.nasa.gov/docs/legacy/CMAPSSData.zip
- Veri açıklama linki: https://data.nasa.gov/dataset/cmapss-jet-engine-simulated-data

### Dataset Details
- Sayısal Veriler: train_FD001.txt, test_FD001.txt ve diğerleri. Bunları PyTorch modelinde kullanılacak.
  - <b>train_FDxxx.txt</b>: Modeli eğitmek için kullanacağız. (1_preproces.ipynb: RUL sütunu çıkarılmıştır)
  - <b>test_FDxxx.txt</b>: Modelin performansını ölçmek için.
  - <b>RUL_FDxxx.txt</b>: Test setindeki motorların gerçek kalan ömür değerlerini içerir (Doğrulama için).

- Dökümantasyon için PDF dosyası: Bu PDF'i Qdrant (Vektör Veri Tabanı) içine gömeceğiz.
  - Senaryo: PyTorch modelin "Bu motorun ömrü 10 döngü kaldı, arıza riski var!" dediğinde, LangGraph ajanı gidip bu PDF'ten "Motor arıza belirtileri ve bakım adımları" dökümanını okuyup kullanıcıya rapor sunacak.

## TechStack
- ML Foundations: PyTorch (Motor sağlığı için LSTM veya derin öğrenme tabanlı regresyon)
- GenAI & Agents: LangGraph (Döngüsel ajan akışları ve hata düzeltme süreçleri için)
- Data & Storage: Qdrant (Vektör veri tabanı - döküman araması için)
- MLOps: MLflow (Deney takibi ve model versiyonlama)
- Integration: FastAPI (Hızlı ve modern REST API)
- Frontend: Streamlit (Hızlı web arayüzü oluşturmak için)

## Project Folder Structure
- /notebooks: PyTorch ile model eğitimi ve MLflow ile kaydedilen metrikler.
- /ingestion: Teknik PDF'lerin parçalanıp Qdrant'a yüklenme scriptleri.
- /src/agent: LangGraph akış şeması (StateGraph tanımları).
- /src/api: FastAPI endpointleri.
- /frontend: Streamlit dashboard kodu.
- docker-compose.yml: Tüm sistemi (Qdrant, API, Frontend) tek tuşla ayağa kaldırmak için.

## Topics used in the Project
### 🔗 LangChain: Standart Zincirleme (Chains)
- LangChain, dil modellerini (LLM) diğer araçlara veya veri kaynaklarına bağlamak için kullanılan temel kütüphanedir. Genellikle DAG (Directed Acyclic Graph - Tek Yönlü Döngüsüz Grafik) mantığıyla çalışır.
- Ne İşe Yarar? Bir girdiyi alır, belirli adımlardan geçirir ve çıktı üretir. (Örn: Soru -> Döküman Bul -> Yanıtla).
- Temel Sorunu: Gerçek dünyadaki iş akışları her zaman dümdüz ilerlemez. Bir hata oluştuğunda geri dönmek veya bir döngüye girip işlemi tekrarlamak LangChain’in standart "Chain" yapısıyla oldukça zordur.
- En İyi Kullanım Alanı: Basit RAG (Retrieval Augmented Generation) sistemleri ve doğrusal veri işleme boru hatları.

### 🕸️ LangGraph: Döngüsel Ajanlar (Stateful Agents)
- LangGraph, LangChain üzerine inşa edilmiş, döngüsel (cyclic) grafikler oluşturmanıza olanak tanıyan bir kütüphanedir. İş akışına "durum" (state) ve "bellek" (persistence) kavramlarını ekler.
- Ne İşe Yarar? Karmaşık, kendi kendine karar verebilen ve gerektiğinde bir önceki adıma geri dönebilen Ajanlar (Agents) yapmanı sağlar.
- Döngüler (Cycles): Bir ajan, araçtan gelen cevabı beğenmezse tekrar deneyebilir veya eksik bilgi için başka bir dala sapabilir.
- Durum Yönetimi (State): Tüm konuşma veya işlem geçmişini bir değişken içinde tutar ve her düğüm (node) bu durumu güncelleyebilir.
- İnsan Müdahalesi (Human-in-the-loop): Kritik bir adımda durup insandan onay alıp sonra devam edebilir.
- En İyi Kullanım Alanı: Hata ayıklama yapan asistanlar, sistem izleme araçları veya çok adımlı analiz gerektiren otonom ajanlar.

## Step-by-step
### 1. Kalan kullanım ömrü
- Bir havacılık projesinde en kritik nokta, ham sensör verisinden "uçağın motoru ne zaman bozulacak?" (Remaining Useful Life - RUL) sorusuna yanıt verebilmektir.

### 2. Kalan kullanım ömrü

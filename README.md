# BizBot - AI Business Analysis Assistant

## Gambaran Umum

BizBot adalah asisten analisis bisnis berbasis AI yang dirancang khusus untuk memberikan konsultasi dan analisis bisnis yang profesional. Sistem ini memanfaatkan teknologi AI modern melalui OpenRouter API untuk menyediakan wawasan bisnis yang akurat, strategis, dan berbasis data.

## Fitur Utama

- **Analisis Bisnis Komprehensif**: Memberikan analisis mendalam tentang berbagai aspek bisnis
- **Konsultasi Strategi**: Bantuan dalam pengembangan strategi pemasaran dan pertumbuhan bisnis
- **Riset Pasar**: Analisis tren pasar dan kompetitif
- **Pengembangan Produk**: Konsultasi pengembangan produk dan layanan
- **Analisis Profitabilitas**: Evaluasi kinerja keuangan dan profitabilitas bisnis
- **Bahasa Indonesia Formal**: Respons dalam Bahasa Indonesia yang profesional dan mudah dipahami

## Arsitektur Sistem

```
bizbot/
├── backend/
│   ├── app/
│   │   ├── main.py              # Aplikasi FastAPI utama
│   │   ├── router.py            # Definisi endpoint API
│   │   ├── llm_client.py        # Klien untuk OpenRouter API
│   │   ├── business_analyzer.py # Modul analisis bisnis khusus
│   │   └── requirements.txt     # Dependencies Python
│   ├── .env.example            # Template konfigurasi environment
│   └── Dockerfile              # Konfigurasi container backend
└── frontend/
    ├── index.php               # Antarmuka web PHP
    └── dashboard.html          # Dashboard analisis bisnis
```

## Teknologi yang Digunakan

### Backend
- **FastAPI**: Framework web modern untuk Python
- **OpenRouter API**: Gateway untuk berbagai model LLM
- **Uvicorn**: ASGI server untuk menjalankan FastAPI
- **Python-dotenv**: Manajemen environment variables

### Frontend
- **PHP**: Server-side scripting untuk antarmuka web
- **HTML/CSS**: Struktur dan styling halaman web
- **JavaScript**: Interaktivitas pada dashboard
- **Chart.js**: Visualisasi data bisnis

## Persyaratan Sistem

### Untuk Development
- Python 3.11+
- PHP 7.4+
- Docker (opsional, untuk containerization)

### Dependencies Python
- fastapi
- uvicorn[standard]
- requests
- python-dotenv
- pydantic

## Instalasi dan Setup

### 1. Setup Environment

```bash
# Buat direktori project
mkdir bizbot && cd bizbot
mkdir -p backend/app frontend
```

### 2. Konfigurasi Backend

#### File Environment
Buat file `backend/.env`:
```env
OPENROUTER_API_KEY=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
OPENROUTER_MODEL=nvidia/nemotron-nano-12b-v2-vl:free
```

#### Requirements
Buat file `backend/app/requirements.txt`:
```txt
fastapi
uvicorn[standard]
requests
python-dotenv
pydantic
```

### 3. Implementasi Kode

#### File LLM Client (`backend/app/llm_client.py`)
```python
import os
import requests
from dotenv import load_dotenv

load_dotenv()

OPENROUTER_URL = "https://openrouter.ai/api/v1/chat/completions"
API_KEY = os.environ.get("OPENROUTER_API_KEY")

def call_openrouter(system_prompt, user_prompt, model=None):
    if not API_KEY:
        raise RuntimeError("OPENROUTER_API_KEY not set")

    # Sistem prompt fokus pada analisis bisnis
    business_system_prompt = (
        "Kamu adalah BizBot, asisten analisis bisnis yang profesional. "
        "Tugasmu adalah memberikan jawaban yang akurat, logis, dan berbasis data. "
        "Fokus pada topik seperti analisis bisnis, pemasaran, strategi pertumbuhan, "
        "riset pasar, pengembangan produk, profitabilitas, dan konsultasi bisnis lainnya. "
        "Struktur jawaban Anda harus jelas dan sistematis. "
        "Gunakan poin-poin penting dan berikan rekomendasi yang dapat ditindaklanjuti. "
        "Jawablah selalu dalam Bahasa Indonesia yang formal, sistematis, dan mudah dipahami."
    )

    headers = {
        "Authorization": f"Bearer {API_KEY}",
        "Content-Type": "application/json",
    }

    payload = {
        "model": model or os.environ.get("OPENROUTER_MODEL", "nvidia/nemotron-nano-12b-v2-vl:free"),
        "messages": [
            {"role": "system", "content": business_system_prompt},
            {"role": "user", "content": user_prompt},
        ],
        "temperature": 0.25
    }

    try:
        r = requests.post(OPENROUTER_URL, json=payload, headers=headers)
        r.raise_for_status()
        data = r.json()
        return data["choices"][0]["message"]["content"]
    except requests.exceptions.RequestException as e:
        return f"Error dalam menghubungi API: {str(e)}"
    except KeyError:
        return "Error: Format respons tidak sesuai yang diharapkan"
```

#### File Business Analyzer (`backend/app/business_analyzer.py`)
```python
class BusinessAnalyzer:
    def __init__(self):
        self.analysis_templates = {
            "swot": {
                "strengths": "Analisis kekuatan internal perusahaan",
                "weaknesses": "Analisis kelemahan internal perusahaan", 
                "opportunities": "Analisis peluang eksternal",
                "threats": "Analisis ancaman eksternal"
            },
            "market_analysis": {
                "size": "Ukuran pasar dan pertumbuhan",
                "competition": "Analisis persaingan",
                "trends": "Tren industri terkini",
                "segmentation": "Segmentasi pasar"
            },
            "financial": {
                "profitability": "Analisis profitabilitas",
                "cash_flow": "Analisis arus kas",
                "investment": "Analisis investasi",
                "valuation": "Penilaian bisnis"
            }
        }
    
    def get_analysis_framework(self, analysis_type):
        return self.analysis_templates.get(analysis_type, {})
```

#### File Router (`backend/app/router.py`)
```python
from fastapi import APIRouter
from pydantic import BaseModel
from llm_client import call_openrouter
from business_analyzer import BusinessAnalyzer

router = APIRouter()
analyzer = BusinessAnalyzer()

class ChatRequest(BaseModel):
    message: str
    analysis_type: str = "general"

class AnalysisResponse(BaseModel):
    analysis: str
    recommendations: list
    confidence: float

@router.post("/analyze", response_model=AnalysisResponse)
async def analyze_business(request: ChatRequest):
    """
    Endpoint untuk analisis bisnis komprehensif
    """
    user_prompt = f"""
    Permintaan analisis bisnis: {request.message}
    Tipe analisis: {request.analysis_type}
    
    Berikan analisis yang mendalam dan rekomendasi yang dapat ditindaklanjuti.
    """
    
    analysis_result = call_openrouter("", user_prompt)
    
    return AnalysisResponse(
        analysis=analysis_result,
        recommendations=[
            "Implementasi strategi berdasarkan analisis",
            "Monitoring dan evaluasi berkala",
            "Penyesuaian berdasarkan feedback pasar"
        ],
        confidence=0.85
    )

@router.post("/consult")
async def business_consultation(request: ChatRequest):
    """
    Endpoint untuk konsultasi bisnis umum
    """
    response = call_openrouter("", request.message)
    return {"consultation": response}

@router.get("/frameworks/{framework_type}")
async def get_analysis_framework(framework_type: str):
    """
    Endpoint untuk mendapatkan template analisis bisnis
    """
    framework = analyzer.get_analysis_framework(framework_type)
    return {"framework": framework}
```

#### File Main Application (`backend/app/main.py`)
```python
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from router import router

app = FastAPI(
    title="BizBot API",
    description="AI Business Analysis Assistant",
    version="1.0.0"
)

# Configure CORS
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

app.include_router(router, prefix="/api/v1")

@app.get("/")
async def root():
    return {
        "message": "BizBot Business Analysis API", 
        "version": "1.0.0",
        "endpoints": {
            "analyze": "/api/v1/analyze",
            "consult": "/api/v1/consult", 
            "frameworks": "/api/v1/frameworks/{type}"
        }
    }

@app.get("/health")
async def health_check():
    return {"status": "healthy", "service": "BizBot"}
```

### 4. Frontend Implementation

#### File Dashboard (`frontend/dashboard.html`)
```html
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>BizBot - Business Analysis Dashboard</title>
    <style>
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            margin: 0;
            padding: 20px;
            background-color: #f5f5f5;
        }
        .container {
            max-width: 1200px;
            margin: 0 auto;
            background: white;
            padding: 30px;
            border-radius: 10px;
            box-shadow: 0 2px 10px rgba(0,0,0,0.1);
        }
        .header {
            text-align: center;
            margin-bottom: 30px;
            border-bottom: 2px solid #2c3e50;
            padding-bottom: 20px;
        }
        .analysis-form {
            margin-bottom: 30px;
        }
        .form-group {
            margin-bottom: 20px;
        }
        label {
            display: block;
            margin-bottom: 5px;
            font-weight: bold;
            color: #2c3e50;
        }
        textarea, select {
            width: 100%;
            padding: 10px;
            border: 1px solid #ddd;
            border-radius: 5px;
            font-size: 14px;
        }
        button {
            background-color: #2c3e50;
            color: white;
            padding: 12px 30px;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            font-size: 16px;
        }
        button:hover {
            background-color: #34495e;
        }
        .result {
            margin-top: 30px;
            padding: 20px;
            background-color: #f8f9fa;
            border-radius: 5px;
            border-left: 4px solid #2c3e50;
        }
        .framework-cards {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
            gap: 20px;
            margin-top: 20px;
        }
        .card {
            background: white;
            padding: 20px;
            border-radius: 8px;
            box-shadow: 0 2px 5px rgba(0,0,0,0.1);
        }
    </style>
</head>
<body>
    <div class="container">
        <div class="header">
            <h1>BizBot - Business Analysis Assistant</h1>
            <p>Asisten AI profesional untuk analisis dan konsultasi bisnis</p>
        </div>

        <div class="analysis-form">
            <h2>Analisis Bisnis</h2>
            <form id="analysisForm">
                <div class="form-group">
                    <label for="analysisType">Jenis Analisis:</label>
                    <select id="analysisType" name="analysisType">
                        <option value="general">Konsultasi Umum</option>
                        <option value="swot">Analisis SWOT</option>
                        <option value="market_analysis">Analisis Pasar</option>
                        <option value="financial">Analisis Keuangan</option>
                        <option value="strategy">Strategi Bisnis</option>
                    </select>
                </div>
                <div class="form-group">
                    <label for="businessQuery">Pertanyaan atau Permintaan Analisis:</label>
                    <textarea id="businessQuery" name="businessQuery" rows="6" 
                              placeholder="Jelaskan bisnis Anda, tantangan yang dihadapi, atau pertanyaan spesifik..."></textarea>
                </div>
                <button type="submit">Dapatkan Analisis</button>
            </form>
        </div>

        <div id="result" class="result" style="display: none;">
            <h3>Hasil Analisis</h3>
            <div id="analysisContent"></div>
        </div>

        <div class="framework-cards">
            <div class="card">
                <h3>Analisis SWOT</h3>
                <p>Identifikasi Strengths, Weaknesses, Opportunities, Threats</p>
            </div>
            <div class="card">
                <h3>Analisis Pasar</h3>
                <p>Riset kompetitif dan segmentasi pasar</p>
            </div>
            <div class="card">
                <h3>Analisis Keuangan</h3>
                <p>Evaluasi profitabilitas dan kesehatan finansial</p>
            </div>
        </div>
    </div>

    <script>
        document.getElementById('analysisForm').addEventListener('submit', async function(e) {
            e.preventDefault();
            
            const analysisType = document.getElementById('analysisType').value;
            const businessQuery = document.getElementById('businessQuery').value;
            
            if (!businessQuery.trim()) {
                alert('Silakan masukkan pertanyaan atau permintaan analisis');
                return;
            }

            try {
                const response = await fetch('http://localhost:8000/api/v1/analyze', {
                    method: 'POST',
                    headers: {
                        'Content-Type': 'application/json',
                    },
                    body: JSON.stringify({
                        message: businessQuery,
                        analysis_type: analysisType
                    })
                });

                const data = await response.json();
                
                document.getElementById('analysisContent').innerHTML = 
                    `<div style="white-space: pre-wrap;">${data.analysis}</div>
                     <h4>Rekomendasi:</h4>
                     <ul>${data.recommendations.map(rec => `<li>${rec}</li>`).join('')}</ul>`;
                
                document.getElementById('result').style.display = 'block';
                
            } catch (error) {
                console.error('Error:', error);
                alert('Terjadi kesalahan saat memproses permintaan');
            }
        });
    </script>
</body>
</html>
```

### 5. Menjalankan Aplikasi

#### Menjalankan Backend
```bash
cd backend/app
pip install -r requirements.txt
uvicorn main:app --reload --host 0.0.0.0 --port 8000
```

#### Menjalankan Frontend
```bash
cd frontend
# Untuk PHP interface
php -S localhost:8080

# Atau buka langsung file HTML
open dashboard.html
```

## API Endpoints

### POST /api/v1/analyze
Analisis bisnis komprehensif dengan rekomendasi terstruktur.

**Request:**
```json
{
  "message": "Saya ingin menganalisis potensi pasar untuk produk makanan organik",
  "analysis_type": "market_analysis"
}
```

**Response:**
```json
{
  "analysis": "Analisis mendalam tentang pasar...",
  "recommendations": [
    "Fokus pada segmentasi pasar millennial",
    "Kembangkan strategi digital marketing",
    "Buat diferensiasi produk yang jelas"
  ],
  "confidence": 0.85
}
```

### POST /api/v1/consult
Konsultasi bisnis umum.

### GET /api/v1/frameworks/{type}
Mendapatkan template analisis bisnis.

## Use Cases

### 1. Analisis Pasar
- Riset kompetitif
- Segmentasi pasar
- Tren industri
- Peluang pertumbuhan

### 2. Strategi Bisnis
- Perencanaan strategis
- Pengembangan produk
- Ekspansi bisnis
- Inovasi model bisnis

### 3. Analisis Keuangan
- Evaluasi profitabilitas
- Manajemen arus kas
- Analisis investasi
- Perencanaan keuangan

### 4. Konsultasi Manajemen
- Optimasi operasional
- Manajemen tim
- Pengambilan keputusan
- Manajemen risiko

## Keamanan dan Best Practices

1. **Environment Variables**: Selalu simpan API key dalam environment variables
2. **Error Handling**: Implementasi error handling yang robust
3. **Rate Limiting**: Pertimbangkan implementasi rate limiting untuk API
4. **Validation**: Validasi input pengguna secara ketat
5. **Monitoring**: Setup monitoring untuk performa dan error tracking

## Pengembangan Lanjutan

### Fitur yang Dapat Ditambahkan
- Integrasi dengan database bisnis
- Analytics dashboard yang lebih kompleks
- Export laporan analisis
- Multi-user support
- Integration dengan tools bisnis eksternal

### Optimasi
- Caching untuk response yang sering digunakan
- Async processing untuk analisis kompleks
- Load balancing untuk traffic tinggi
- Database persistence untuk history analisis

BizBot dirancang untuk menjadi asisten analisis bisnis yang powerful dan mudah diintegrasikan ke dalam berbagai workflow bisnis modern.

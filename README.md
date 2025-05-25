# Tutorial Continuous Code Inspection dengan SonarCloud

## Ringkasan Singkat
Tutorial ini menjelaskan cara melakukan continuous code inspection menggunakan SonarCloud untuk menganalisis kualitas kode Java. Kita akan setup SonarCloud, mengintegrasikan dengan GitHub Actions, menganalisis issues yang ditemukan, dan memperbaiki kode untuk meningkatkan kualitas.

## 1. Persiapan Awal

### Setup Repository
1. **Buat repository baru di GitHub**
   - Login ke GitHub
   - Klik "New repository"
   - Beri nama repository (contoh: "java-counter-sonarcloud")
   - Set sebagai public repository
   - Inisialisasi dengan README

2. **Upload file Java**
   - Buat struktur folder sesuai package:
     ```
     src/
     ├── main/
     │   └── java/
     │       └── Counter.java
     └── test/
         └── java/
             └── CounterTest.java
     Driver.java
     ```

## 2. Setup SonarCloud (Langkah Singkat)

1. **Login ke SonarCloud** → https://sonarcloud.io dengan akun GitHub
2. **Import Repository** → Klik "+" → "Analyze new project" → Pilih repo
3. **Generate Token** → Copy token untuk GitHub Secrets
4. **Set GitHub Secret** → Repository Settings → Secrets → Add `SONAR_TOKEN`

## 3. File Konfigurasi (Copy-Paste Ready)

### A. Buat `.github/workflows/sonarcloud.yml`
```yaml
name: SonarCloud Analysis
on:
  push:
    branches: [ main, master ]
  pull_request:
    branches: [ main, master ]
jobs:
  sonarcloud:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
      with:
        fetch-depth: 0
    - name: Set up JDK 11
      uses: actions/setup-java@v3
      with:
        java-version: '11'
        distribution: 'temurin'
    - name: SonarCloud Scan
      uses: sonarSource/sonarcloud-github-action@master
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
```

### B. Buat `sonar-project.properties`
```properties
sonar.organization=your-github-username
sonar.projectKey=your-github-username_repository-name
sonar.projectName=Java Counter Analysis
sonar.sources=src/main/java,Driver.java
sonar.tests=src/test/java
sonar.language=java
sonar.sourceEncoding=UTF-8
sonar.java.source=8
```

## 4. Issues yang Akan Ditemukan & Cara Fix

### 🐛 **BUG - Method powerBy() (CRITICAL)**
```java
// ❌ SALAH (menggunakan XOR):
public void powerBy(int i){
    count = count ^ i;
}

// ✅ BENAR (operasi pangkat):
public void powerBy(int i){
    count = (int) Math.pow(count, i);
}
```

### 🔧 **CODE SMELL - Method triple()**
```java
// ❌ Variable tidak perlu:
public void triple(){
    int i = 3;
    multiplyBy(i);
}

// ✅ Gunakan konstanta:
private static final int TRIPLE_FACTOR = 3;

public void triple(){
    multiplyBy(TRIPLE_FACTOR);
}
```

### 📦 **MINOR - Package Declaration**
```java
// ❌ Driver.java tanpa package:
import src.main.java.Counter;
public class Driver { ... }

// ✅ Tambahkan package:
package src.main.java;
import src.main.java.Counter;
public class Driver { ... }
```

## 5. Langkah Eksekusi (Step-by-Step)

### Tahap 1: Setup Repository
1. Buat repository GitHub baru (public)
2. Upload file Java dengan struktur folder yang benar
3. Commit dan push initial code

### Tahap 2: Konfigurasi SonarCloud
1. Login ke sonarcloud.io dengan GitHub
2. Import repository yang dibuat
3. Generate dan copy token
4. Tambahkan token ke GitHub Secrets (`SONAR_TOKEN`)

### Tahap 3: Setup Files
1. Buat workflow file (copy dari contoh di atas)
2. Buat sonar-project.properties (sesuaikan organization/project key)
3. Commit kedua file ini

### Tahap 4: Analisis Pertama
1. Push/commit akan trigger GitHub Actions
2. Tunggu scan selesai (2-3 menit)
3. Buka SonarCloud dashboard untuk lihat hasil

### Tahap 5: Perbaikan Code
1. Fix issues satu per satu (lihat contoh di atas)
2. Commit setiap perbaikan
3. Monitor improvement di dashboard

## 6. Memahami Dashboard SonarCloud

### 📊 **Main Metrics**
- **🐛 Bugs**: Issues yang bisa menyebabkan error (harus diperbaiki)
- **🔒 Vulnerabilities**: Security issues (prioritas tinggi)  
- **💡 Code Smells**: Maintainability issues (bisa diperbaiki bertahap)
- **📈 Coverage**: Persentase code yang ter-test
- **📋 Duplications**: Duplicate code blocks

### 🎯 **Quality Gate**
- **PASSED**: Code quality memenuhi standar minimum
- **FAILED**: Ada critical issues yang harus diperbaiki
- Quality gate bisa dikustomisasi sesuai standar team

### 📈 **Rating System (A-E)**
- **A**: Excellent (0 issues)
- **B**: Good (minor issues)  
- **C**: Acceptable (beberapa issues)
- **D**: Poor (many issues)
- **E**: Bad (critical issues)

## 7. Tips untuk Video Recording

### 🎥 **Struktur Video yang Disarankan**
1. **Opening** (2-3 min): Perkenalan dan tujuan
2. **Setup** (5-7 min): GitHub repo, SonarCloud config  
3. **First Analysis** (5-8 min): Review issues yang ditemukan
4. **Code Fixes** (8-10 min): Perbaiki issues dan show improvement
5. **Dashboard Tour** (3-5 min): Explain metrics dan monitoring
6. **Closing** (2 min): Summary dan benefits

### 📋 **Checklist Sebelum Recording**
- [ ] Repository GitHub sudah ready dengan files
- [ ] SonarCloud account sudah setup
- [ ] Browser tabs sudah disiapkan (GitHub + SonarCloud)
- [ ] Code editor ready untuk demo fixes
- [ ] Internet connection stable
- [ ] Webcam dan audio test

### 💡 **Speaking Tips**
- Explain **why** not just **what** you're doing
- Zoom in saat show code details
- Pause sejenak saat menunggu build/scan
- Show enthusiasm tentang code quality improvement
- Make eye contact dengan camera

## 8. Quick Reference & Troubleshooting

### 🔧 **Common Issues & Solutions**
| Problem | Solution |
|---------|----------|
| Build gagal di GitHub Actions | Check SONAR_TOKEN di repository secrets |
| "Organization not found" | Pastikan organization name di sonar-project.properties benar |
| No coverage data | Tambahkan JaCoCo plugin atau exclude coverage |
| Workflow tidak trigger | Check branch name di workflow file (main vs master) |

### 📚 **Key Commands untuk Demo**
```bash
# Clone repository
git clone https://github.com/username/repository-name.git

# Add files
git add .
git commit -m "Add SonarCloud configuration"
git push origin main

# Check GitHub Actions
# Go to repository → Actions tab → lihat running workflow
```

### 🎯 **Expected Results After Setup**
- ✅ GitHub Actions workflow running successfully
- ✅ SonarCloud dashboard showing project analysis
- ✅ Issues detected: ~3-4 bugs/code smells
- ✅ Quality Gate status (likely FAILED initially)
- ✅ After fixes: Quality Gate PASSED

---

## Summary
Tutorial ini memberikan panduan lengkap untuk implementasi continuous code inspection dengan SonarCloud. Setelah mengikuti tutorial ini, Anda akan memiliki automated code quality monitoring yang terintegrasi dengan GitHub workflow, serta pemahaman tentang cara mengidentifikasi dan memperbaiki issues dalam kode Java.

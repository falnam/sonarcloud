# Panduan Continuous Code Inspection dengan SonarCloud

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

## 2. Setup SonarCloud

### Registrasi dan Konfigurasi
1. **Akses SonarCloud**
   - Buka https://sonarcloud.io
   - Login menggunakan akun GitHub
   - Authorize SonarCloud untuk mengakses repository

2. **Import Repository**
   - Klik "+" di pojok kanan atas
   - Pilih "Analyze new project"
   - Pilih organization GitHub Anda
   - Select repository yang telah dibuat
   - Klik "Set up"

3. **Konfigurasi Project**
   - Project key akan otomatis generate
   - Set project name sesuai keinginan
   - Pilih "Previous version" untuk baseline

## 3. Konfigurasi Build

### Setup untuk Java Project
1. **Pilih Analysis Method**
   - Pilih "With GitHub Actions" (recommended)
   - Atau "With other CI tools" jika menggunakan CI/CD lain

2. **Generate Token**
   - SonarCloud akan generate token otomatis
   - Copy token untuk digunakan di GitHub Secrets

3. **Setup GitHub Secrets**
   - Di repository GitHub, masuk ke Settings > Secrets and variables > Actions
   - Klik "New repository secret"
   - Name: `SONAR_TOKEN`
   - Value: paste token dari SonarCloud

## 4. Konfigurasi Workflow File

### Buat GitHub Actions Workflow
Buat file `.github/workflows/sonarcloud.yml`:

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
        # Disabling shallow clone is recommended for improving relevancy of reporting
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

### Buat sonar-project.properties
Buat file `sonar-project.properties` di root repository:

```properties
# Organization and project keys
sonar.organization=your-github-username
sonar.projectKey=your-github-username_java-counter-sonarcloud

# Metadata
sonar.projectName=Java Counter SonarCloud
sonar.projectVersion=1.0

# Source code
sonar.sources=src/main/java,Driver.java
sonar.tests=src/test/java

# Language
sonar.language=java

# Encoding
sonar.sourceEncoding=UTF-8

# Java version
sonar.java.source=8
sonar.java.target=8

# Coverage exclusions
sonar.coverage.exclusions=**/*Test.java,**/Driver.java
```

## 5. Analisis Code Issues

### Issues yang Akan Ditemukan di Code
Berdasarkan file Java yang diberikan, SonarCloud kemungkinan akan mendeteksi:

#### Counter.java Issues:
1. **Bug di method `powerBy(int i)`**
   ```java
   public void powerBy(int i){
       count = count ^ i;  // Ini XOR, bukan power!
   }
   ```
   - Seharusnya menggunakan `Math.pow()` untuk operasi pangkat

2. **Code Smell - Unused variable**
   ```java
   public void triple(){
       int i = 3;  // Variable bisa langsung digunakan
       multiplyBy(i);
   }
   ```

3. **Code Smell - Magic Number**
   - Angka 3 di method `triple()` sebaiknya dijadikan konstanta

#### Driver.java Issues:
1. **Missing package declaration**
2. **System.out.println concatenation** - sebaiknya menggunakan formatting

#### CounterTest.java Issues:
1. **Assertion order** - actual dan expected mungkin terbalik
2. **Missing edge case tests**

## 6. Simulasi Perbaikan Code

### Langkah Perbaikan:
1. **Fix Bug di powerBy method**:
   ```java
   public void powerBy(int i){
       count = (int) Math.pow(count, i);
   }
   ```

2. **Improve triple method**:
   ```java
   private static final int TRIPLE_FACTOR = 3;
   
   public void triple(){
       multiplyBy(TRIPLE_FACTOR);
   }
   ```

3. **Fix Driver.java**:
   ```java
   package src.main.java;
   import src.main.java.Counter;

   public class Driver {
       public static void main(String[] args) {
           Counter counter = new Counter();
           
           System.out.printf("Current count: %d%n", counter.getCount());
           counter.increment();
           System.out.printf("Current count: %d%n", counter.getCount());
           counter.decrement();
           System.out.printf("Current count: %d%n", counter.getCount());
       }
   }
   ```

## 7. Monitoring dan Continuous Inspection

### Dashboard SonarCloud
1. **Quality Gate Status** - Pass/Fail status
2. **Bugs** - Jumlah bug yang ditemukan
3. **Vulnerabilities** - Security issues
4. **Code Smells** - Maintainability issues
5. **Coverage** - Test coverage percentage
6. **Duplications** - Duplicate code blocks

### Metrics yang Dimonitor:
- **Reliability Rating** (A-E)
- **Security Rating** (A-E)
- **Maintainability Rating** (A-E)
- **Test Coverage** (%)
- **Duplicated Lines** (%)

## 8. Best Practices

### Untuk Continuous Inspection:
1. **Set Quality Gate** yang sesuai dengan standar tim
2. **Review hasil scan** setiap push/PR
3. **Prioritize Critical dan High severity issues**
4. **Maintain coverage** minimal 80%
5. **Fix issues** sebelum merge ke main branch

### Tips Recording Video:
1. **Persiapkan environment** - browser, IDE, terminal
2. **Explain each step** yang dilakukan
3. **Show before/after** code quality metrics
4. **Demonstrate** fix untuk beberapa issues
5. **Show dashboard** dan cara interpretasi results

## 9. Checklist untuk Video Recording

- [ ] Setup SonarCloud account dan import repository
- [ ] Konfigurasi GitHub Actions workflow
- [ ] Upload initial code dan trigger first scan
- [ ] Review dan explain issues yang ditemukan
- [ ] Demonstrate code fixes
- [ ] Show improved metrics after fixes
- [ ] Explain continuous monitoring process
- [ ] Show Quality Gate configuration
- [ ] Demonstrate PR decoration (jika ada)

## 10. Troubleshooting Common Issues

### Jika Build Gagal:
- Check GitHub Actions logs
- Verify SONAR_TOKEN di repository secrets
- Pastikan sonar-project.properties configuration benar
- Check Java version compatibility

### Jika No Coverage:
- Tambahkan JaCoCo plugin untuk coverage
- Update workflow untuk generate coverage report
- Configure sonar.coverage.jacoco.xmlReportPaths

Dengan mengikuti panduan ini, Anda akan dapat mendemonstrasikan complete workflow dari setup hingga continuous monitoring menggunakan SonarCloud.

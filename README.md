# Tutorial Lengkap: Continuous Code Inspection dengan SonarCloud

## Persiapan Awal

### Yang Dibutuhkan:
- Akun GitHub
- Project Java Maven (folder BASIC-JAVA-MAVEN-MAIN)
- Browser untuk akses SonarCloud
- Software perekam layar (OBS, Bandicam, dll)

---

## BAGIAN 1: Setup Repository di GitHub

### Langkah 1: Upload Project ke GitHub
1. **Buka GitHub.com** dan login ke akun Anda
2. **Klik tombol "New"** untuk membuat repository baru
3. **Isi detail repository:**
   - Repository name: `java-maven-sonar-demo`
   - Description: `Java Maven project for SonarCloud demonstration`
   - Pilih **Public** (agar SonarCloud bisa akses gratis)
   - Centang **Add README file**
   - Pilih **.gitignore template: Java**
4. **Klik "Create repository"**

### Langkah 2: Upload File Project
1. **Klik "uploading an existing file"** atau drag & drop
2. **Upload seluruh folder** BASIC-JAVA-MAVEN-MAIN
3. **Pastikan struktur folder seperti ini:**
   ```
   java-maven-sonar-demo/
   ├── .github/workflows/
   ├── src/
   │   ├── main/java/
   │   └── test/java/
   ├── pom.xml
   ├── .gitignore
   └── README.md
   ```
4. **Commit dengan message:** "Initial commit: Java Maven project"

---

## BAGIAN 2: Setup SonarCloud Account

### Langkah 3: Buat Akun SonarCloud
1. **Buka browser baru** dan kunjungi [https://sonarcloud.io](https://sonarcloud.io)
2. **Klik "Log in"** di pojok kanan atas
3. **Pilih "Log in with GitHub"**
4. **Authorize SonarCloud** untuk mengakses akun GitHub Anda
5. **Tunggu proses redirect** kembali ke SonarCloud

### Langkah 4: Import Repository
1. **Setelah login**, klik tombol **"+"** di pojok kanan atas
2. **Pilih "Analyze new project"**
3. **Pilih "Import from GitHub"**
4. **Cari dan pilih repository** `java-maven-sonar-demo`
5. **Klik "Set up"**

### Langkah 5: Konfigurasi Project
1. **Organization:** akan otomatis terisi dengan username GitHub Anda
2. **Project Key:** biarkan default atau ubah sesuai keinginan
3. **Pilih "With GitHub Actions"** (recommended untuk CI/CD)
4. **Klik "Continue"**

---

## BAGIAN 3: Setup GitHub Actions Integration

### Langkah 6: Copy SONAR_TOKEN
1. **SonarCloud akan menampilkan SONAR_TOKEN**
2. **Copy token tersebut** (hanya ditampilkan sekali, jadi hati-hati!)
3. **Jangan close tab ini dulu**

### Langkah 7: Setup GitHub Secrets
1. **Buka tab baru** dan masuk ke repository GitHub Anda
2. **Klik tab "Settings"** di repository
3. **Pilih "Secrets and variables"** → **"Actions"** di sidebar kiri
4. **Klik "New repository secret"**
5. **Isi:**
   - Name: `SONAR_TOKEN`
   - Secret: paste token dari SonarCloud tadi
6. **Klik "Add secret"**

### Langkah 8: Buat GitHub Actions Workflow
1. **Di repository GitHub**, klik tab **"Actions"**
2. **Klik "set up a workflow yourself"**
3. **Ganti nama file** menjadi `sonar-analysis.yml`
4. **Copy-paste workflow ini:**

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
    - name: Checkout code
      uses: actions/checkout@v3
      with:
        fetch-depth: 0
        
    - name: Set up JDK 11
      uses: actions/setup-java@v3
      with:
        java-version: '11'
        distribution: 'temurin'
        
    - name: Cache Maven dependencies
      uses: actions/cache@v3
      with:
        path: ~/.m2
        key: ${{ runner.os }}-m2-${{ hashFiles('**/pom.xml') }}
        
    - name: Run tests and SonarCloud analysis
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
      run: |
        mvn clean verify sonar:sonar \
          -Dsonar.projectKey=your-project-key \
          -Dsonar.organization=your-org-key \
          -Dsonar.host.url=https://sonarcloud.io
```

5. **Update project-key dan organization** sesuai dengan setting SonarCloud Anda
6. **Klik "Commit changes"**

---

## BAGIAN 4: Update Konfigurasi Maven

### Langkah 9: Update pom.xml
1. **Edit file pom.xml** di repository
2. **Tambahkan properties SonarCloud:**

```xml
<properties>
    <maven.compiler.source>11</maven.compiler.source>
    <maven.compiler.target>11</maven.compiler.target>
    <sonar.organization>your-org-key</sonar.organization>
    <sonar.projectKey>your-project-key</sonar.projectKey>
    <sonar.host.url>https://sonarcloud.io</sonar.host.url>
</properties>
```

3. **Tambahkan SonarCloud plugin:**

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.sonarsource.scanner.maven</groupId>
            <artifactId>sonar-maven-plugin</artifactId>
            <version>3.9.1.2184</version>
        </plugin>
        <plugin>
            <groupId>org.jacoco</groupId>
            <artifactId>jacoco-maven-plugin</artifactId>
            <version>0.8.8</version>
            <executions>
                <execution>
                    <goals>
                        <goal>prepare-agent</goal>
                    </goals>
                </execution>
                <execution>
                    <id>report</id>
                    <phase>test</phase>
                    <goals>
                        <goal>report</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

4. **Commit changes** dengan message: "Add SonarCloud configuration"

---

## BAGIAN 5: Simulasi Analisa Code

### Langkah 10: Trigger First Analysis
1. **Push/commit akan otomatis trigger** GitHub Actions
2. **Masuk ke tab "Actions"** di GitHub repository
3. **Lihat workflow "SonarCloud Analysis"** sedang berjalan
4. **Tunggu sampai selesai** (biasanya 2-5 menit)

### Langkah 11: Lihat Hasil di SonarCloud
1. **Kembali ke tab SonarCloud**
2. **Refresh halaman** atau masuk ke dashboard project
3. **Lihat hasil analysis:**
   - **Bugs:** masalah yang dapat menyebabkan error
   - **Vulnerabilities:** masalah keamanan
   - **Code Smells:** masalah kualitas code
   - **Coverage:** persentase code yang ter-cover oleh test
   - **Duplications:** code yang duplikat

### Langkah 12: Buat Perubahan Code untuk Demo
1. **Edit salah satu file Java** di src/main/java/
2. **Contoh perubahan yang bisa menimbulkan issues:**

```java
public class Example {
    // Unused variable - akan terdeteksi sebagai code smell
    private String unusedVariable = "test";
    
    // Method tanpa dokumentasi
    public void methodWithoutJavadoc() {
        // Empty catch block - akan terdeteksi sebagai bug
        try {
            int result = 10 / 0;
        } catch (Exception e) {
            // Empty catch - code smell
        }
        
        // Hardcoded password - security vulnerability
        String password = "admin123";
        
        // Duplicate code
        System.out.println("Hello World");
        System.out.println("Hello World");
        System.out.println("Hello World");
    }
    
    // Method dengan complexity tinggi
    public void complexMethod(int x) {
        if (x > 0) {
            if (x < 10) {
                if (x % 2 == 0) {
                    if (x > 5) {
                        System.out.println("Complex logic");
                    }
                }
            }
        }
    }
}
```

3. **Commit dengan message:** "Add code with intentional issues for demo"

### Langkah 13: Analisa Hasil Perubahan
1. **Tunggu GitHub Actions selesai** menganalisa perubahan
2. **Buka SonarCloud dashboard**
3. **Lihat perubahan metrics:**
   - **Issues bertambah**
   - **Quality Gate status** (Pass/Fail)
   - **Detailed breakdown** per kategori issue

---

## BAGIAN 6: Perbaikan dan Monitoring

### Langkah 14: Fix Issues yang Ditemukan
1. **Klik pada issue di SonarCloud** untuk melihat detail
2. **Fix issues satu per satu:**

```java
public class Example {
    private static final Logger LOGGER = LoggerFactory.getLogger(Example.class);
    
    /**
     * Method dengan dokumentasi yang proper
     */
    public void improvedMethod() {
        try {
            int result = 10 / 0;
        } catch (ArithmeticException e) {
            LOGGER.error("Division by zero error", e);
            // Handle error properly
        }
        
        // Use configuration for sensitive data
        String password = System.getenv("PASSWORD");
        
        // Extract to method untuk avoid duplication
        printHelloWorld();
    }
    
    private void printHelloWorld() {
        System.out.println("Hello World");
    }
    
    /**
     * Simplified method dengan reduced complexity
     */
    public void simplifiedMethod(int x) {
        if (isValidRange(x) && isEvenAndGreaterThanFive(x)) {
            System.out.println("Valid input");
        }
    }
    
    private boolean isValidRange(int x) {
        return x > 0 && x < 10;
    }
    
    private boolean isEvenAndGreaterThanFive(int x) {
        return x % 2 == 0 && x > 5;
    }
}
```

3. **Commit fixes:** "Fix SonarCloud issues: remove code smells and vulnerabilities"

### Langkah 15: Monitor Improvement
1. **Tunggu analysis selesai**
2. **Lihat improvement di SonarCloud:**
   - **Issues berkurang**
   - **Quality Gate** menjadi Pass
   - **Maintainability Rating** membaik

---

## BAGIAN 7: Tips untuk Video Recording

### Struktur Video yang Direkomendasikan:
1. **Opening (1-2 menit):**
   - Perkenalan diri
   - Penjelasan tujuan demo
   - Overview SonarCloud

2. **Setup Process (5-7 menit):**
   - Setup GitHub repository
   - Konfigurasi SonarCloud
   - Setup GitHub Actions

3. **Demo Analysis (5-7 menit):**
   - Show initial analysis
   - Buat perubahan code dengan issues
   - Tunjukkan hasil detection

4. **Fixing Issues (3-5 menit):**
   - Fix issues yang ditemukan
   - Show improvement metrics

5. **Closing (1-2 menit):**
   - Kesimpulan manfaat continuous inspection
   - Next steps

### Tips Recording:
- **Gunakan screen recorder** yang bisa show webcam (OBS recommended)
- **Bicara dengan jelas** dan explain setiap langkah
- **Zoom in** saat menunjukkan detail penting
- **Pause sejenak** saat waiting untuk proses (GitHub Actions, SonarCloud analysis)
- **Prepare script** untuk intro dan closing

---

## Troubleshooting Common Issues

### Issue 1: GitHub Actions Fail
- **Check SONAR_TOKEN** sudah benar di GitHub Secrets
- **Pastikan pom.xml** sudah update dengan plugin yang benar
- **Check Java version** di workflow (harus match dengan pom.xml)

### Issue 2: SonarCloud Tidak Menerima Data
- **Verify organization dan project key** di pom.xml
- **Check GitHub Actions logs** untuk error details
- **Pastikan repository visibility** public untuk free tier

### Issue 3: No Coverage Data
- **Tambahkan JaCoCo plugin** di pom.xml
- **Pastikan ada unit tests** di src/test/java/
- **Run `mvn test`** locally untuk verify

---

## Kesimpulan

Setelah mengikuti tutorial ini, Anda akan memiliki:
1. ✅ Repository GitHub dengan Maven project
2. ✅ SonarCloud integration yang berfungsi
3. ✅ Automated code analysis via GitHub Actions
4. ✅ Understanding tentang continuous code inspection
5. ✅ Video demonstration yang complete

**Continuous Code Inspection** membantu maintain kualitas code dengan:
- **Automated detection** issues di setiap commit
- **Consistent quality standards** across team
- **Early prevention** bugs dan vulnerabilities
- **Measurable code quality** metrics

Good luck dengan recording video demo Anda! 🎥

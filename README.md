# Draf Presentasi Detail: Proyek Spell Checker

## Slide 1: Judul

**Analisis dan Implementasi Spell Checker dengan Personalisasi**

*Studi Kasus: Penggunaan Trie dan Binary Search Tree di Java*

**Disusun oleh:** [Nama Anda]
**NIM:** [NIM Anda]

---

## Slide 2: Agenda

1.  **Latar Belakang & Tujuan Proyek**
2.  **Arsitektur & Alur Kerja**
3.  **Detail Struktur Data 1: `Trie.java` (Kamus)**
    -   Struktur & Konsep
    -   Penjelasan Setiap Metode
4.  **Detail Struktur Data 2: `BinarySearchTree.java` (Personalisasi)**
    -   Perbandingan Implementasi (Lama vs. Baru)
    -   Penjelasan Metode Kunci (`insert`, `search`, `find`)
    -   Alasan Refactoring
5.  **Detail Kelas Utama: `SpellCheckerMain.java`**
    -   Penjelasan Setiap Metode & Alur Program
6.  **Demo Aplikasi & Analisis Hasil**
7.  **Kesimpulan & Tanya Jawab**

---
---

## Konsep Umum & Alur Kerja Program

### Peran Kunci Struktur Data
- **`Trie` (Kamus Kata):** Berperan sebagai kamus utama untuk pengecekan ejaan super cepat.
- **`BinarySearchTree` (Mesin Personalisasi):** Berperan menyimpan frekuensi kata yang sering digunakan untuk memberikan saran yang lebih relevan.
- **`SpellCheckerMain` (Orkestrator):** Program utama yang menggabungkan `Trie` dan `BST` untuk memproses input pengguna.

### Alur Kerja Detail
1.  **Startup & Inisialisasi:**
    *   Kamus dari file `.csv` dimuat ke dalam **Trie**.
    *   **BinarySearchTree** kosong (`freqBst`) dibuat untuk melacak frekuensi kata.
2.  **Input & Normalisasi:**
    *   Pengguna memasukkan kalimat (Contoh: "Saya senag belahar bahasa Indonesia.").
    *   Kalimat diubah ke huruf kecil dan dipecah menjadi kata-kata: `["saya", "senag", "belahar", "bahasa", "indonesia"]`.
3.  **Proses Kata per Kata:**
    *   **Untuk Kata Valid (misal, "saya"):**
        *   Ditemukan di `Trie`.
        *   Frekuensinya di-update di `BST` (jika baru jadi 1, jika sudah ada di-increment).
    *   **Untuk Kata Salah Eja (misal, "senag"):**
        *   Tidak ditemukan di `Trie`.
        *   `Trie` menghasilkan kandidat koreksi (misal: `["senang"]`).
        *   Kandidat diurutkan berdasarkan frekuensi dari `BST`.
        *   Saran terbaik ditampilkan. Jika pengguna memilih koreksi, frekuensi kata yang benar di-update di `BST`.
4.  **Personalisasi Berkelanjutan:**
    *   `BST` menjadi "lebih pintar" seiring waktu, memprioritaskan saran berdasarkan riwayat penggunaan.

---

## Batasan Proyek & Ruang Pengembangan

- **Hanya Edit Distance 1:** Tidak bisa memperbaiki typo yang lebih dari satu kesalahan (misal: "blahar" -> "belajar").
- **Tidak Memahami Konteks:** Pengecekan hanya per-kata. Kalimat seperti "Saya makan *masi*" akan dianggap benar jika "masi" ada di kamus, meskipun konteksnya salah.
- **Tidak Menangani Imbuhan/Kata Majemuk:** Tidak bisa mengurai kata seperti "mempertanggungjawabkan" ke kata dasarnya "tanggung jawab" tanpa algoritma *stemming*.

---
---

## BAGIAN 1: TRIE.JAVA (KAMUS UTAMA)

---

## Slide 3: `Trie.java` - Peran & Struktur Dasar

-   **Peran Utama:** Sebagai kamus utama. Menyimpan lebih dari 140.000 kata dari KBBI untuk proses pengecekan ejaan yang sangat cepat.
-   **Struktur `TrieNode`**:
    ```java
    class TrieNode {
        // Menyimpan node anak. Key adalah karakter, Value adalah node berikutnya.
        Map<Character, TrieNode> children = new HashMap<>();
        
        // Flag boolean untuk menandai akhir dari sebuah kata yang valid.
        boolean isEndOfWord; 
    }
    ```
-   **Mengapa `HashMap`?** Kode asli dari asisten dosen mungkin menggunakan array. Kita menggunakan `HashMap` karena:
    1.  **Fleksibel:** Tidak terbatas pada alfabet 'a-z'. Bisa menangani angka, simbol, atau huruf dari bahasa lain.
    2.  **Efisien Memori:** Hanya mengalokasikan memori untuk karakter anak yang benar-benar ada, tidak seperti array yang mengalokasikan 26 slot untuk setiap node.

---

## Slide 4: `Trie.java` - Metode `insert`

### `public void insert(String word)`
-   **Tujuan:** Menambahkan sebuah kata ke dalam kamus Trie.
-   **Cara Kerja:**
    1.  Mulai dari `root`.
    2.  Iterasi setiap karakter dari kata (sudah diubah ke huruf kecil).
    3.  Untuk setiap karakter, periksa apakah path-nya sudah ada di `children` Map.
    4.  Jika belum, buat node baru (`putIfAbsent`).
    5.  Pindah ke node anak tersebut.
    6.  Setelah semua karakter selesai, tandai node terakhir dengan `isEndOfWord = true`.
-   **Kompleksitas Waktu:** O(L), dimana L adalah panjang kata. Sangat efisien.
-   **Kode:**
    ```java
    public void insert(String word) {
        TrieNode current = root;
        for (char ch : word.toLowerCase().toCharArray()) {
            if (!Character.isLetter(ch)) {
                continue; // Abaikan karakter non-huruf
            }
            // Buat node baru jika path belum ada
            current.children.putIfAbsent(ch, new TrieNode());
            // Pindah ke node berikutnya
            current = current.children.get(ch);
        }
        // Tandai akhir dari kata yang valid
        current.isEndOfWord = true;
    }
    ```

---

## Slide 5: `Trie.java` - Metode `contains`

### `public boolean contains(String word)`
-   **Tujuan:** Memeriksa apakah sebuah kata *tepat* ada di dalam kamus.
-   **Cara Kerja:**
    1.  Menggunakan metode pembantu `searchNode(word)` untuk menelusuri path dari kata.
    2.  `searchNode` akan mengembalikan node terakhir dari path jika ada, atau `null` jika path terputus.
    3.  `contains` mengembalikan `true` **hanya jika** `node != null` (path ada) **DAN** `node.isEndOfWord == true` (path tersebut adalah kata yang valid).
-   **Kompleksitas Waktu:** O(L).
-   **Kode:**
    ```java
    public boolean contains(String word) {
        // Cari node terakhir berdasarkan path kata
        TrieNode node = searchNode(word.toLowerCase());
        // Return true HANYA JIKA node-nya ada DAN ditandai sebagai akhir kata
        return node != null && node.isEndOfWord;
    }
    ```

---

## Slide 6: `Trie.java` - Helper `searchNode`

### `private TrieNode searchNode(String word)`
-   **Tujuan:** Ini adalah "mesin pencari" internal di dalam Trie. Tujuannya adalah untuk menavigasi Trie berdasarkan path dari sebuah kata dan mengembalikan node terakhir jika path tersebut ada.

```java
private TrieNode searchNode(String word) {
    // 1. Mulai penelusuran dari root.
    TrieNode current = root;
    // 2. Loop untuk setiap karakter dalam kata.
    for (char ch : word.toCharArray()) {
        // 3. Periksa apakah karakter saat ini ada di children map.
        if(!current.children.containsKey(ch)) {
            // 4. Jika tidak, path terputus. Kata ini tidak ada.
            return null;
        }
        // 5. Jika ada, pindah ke node anak untuk melanjutkan.
        current = current.children.get(ch);
    }
    // 6. Jika loop selesai, kita berhasil menemukan path-nya.
    return current;
}
```

---

## Slide 7: `Trie.java` - Metode `suggest` (Autocomplete)

### `public List<String> suggest(String prefix)`
-   **Tujuan:** Memberikan saran kata-kata yang dimulai dengan awalan (prefix) tertentu. Fitur ini tidak digunakan di SpellChecker, tapi merupakan fungsi standar Trie.

```java
public List<String> suggest(String prefix) {
    List<String> suggestions = new ArrayList<>();
    TrieNode node = root;
    StringBuilder sb = new StringBuilder();

    // LANGKAH 1: Navigasi ke node terakhir dari prefix.
    for (char ch : prefix.toLowerCase().toCharArray()) {
        node = node.children.get(ch);
        if (node == null) {
            // Jika prefix tidak ada, kembalikan list kosong.
            return suggestions;
        }
        sb.append(ch);
    }
    
    // LANGKAH 2: Panggil helper untuk mengumpulkan semua kata
    // di bawah node prefix tersebut.
    collectWords(node, sb, suggestions);
    return suggestions;
}
```

---

## Slide 8: `Trie.java` - Helper `collectWords` (Rekursif)

### `private void collectWords(TrieNode node, StringBuilder currentWord, List<String> suggestions)`
-   **Tujuan:** Menjelajahi semua cabang di bawah sebuah node secara rekursif dan mengumpulkan semua kata yang valid.

```java
private void collectWords(TrieNode node, StringBuilder current, List<String> out) {
    // 1. KONDISI BERHENTI: Jika node ini adalah akhir sebuah kata...
    if (node.isEndOfWord) {
        // ...tambahkan kata yang sudah terbentuk ke dalam daftar hasil.
        out.add(current.toString());
    }

    // 2. LANGKAH REKURSIF: Loop untuk setiap anak dari node saat ini.
    for (Map.Entry<Character, TrieNode> entry : node.children.entrySet()) {
        // a. Tambahkan karakter anak ke kata saat ini.
        current.append(entry.getKey());
        // b. Panggil diri sendiri untuk menjelajahi cabang ini lebih dalam.
        collectWords(entry.getValue(), current, out);
        // c. BACKTRACKING: Hapus karakter terakhir setelah cabang
        //    selesai dijelajahi, agar bisa menjelajahi cabang lain.
        current.deleteCharAt(current.length() - 1);
    }
}
```

---

## Slide 9: Intermezzo - Mengapa menggunakan `StringBuilder`?

-   **Pertanyaan:** Mengapa kita menggunakan `StringBuilder` dan bukan `String` biasa di metode `collectWords`?

-   **Jawaban:** Karena **efisiensi**.

| Fitur | `String` (Biasa) | `StringBuilder` (Yang Kita Pakai) |
| :--- | :--- | :--- |
| **Sifat** | **Immutable** (Tidak Bisa Diubah) | **Mutable** (Bisa Diubah) |
| **Operasi `+`** | Membuat objek **BARU** di memori setiap saat. | Memodifikasi objek yang **SAMA** di memori. |
| **Contoh** | `String s = "a"; s = s + "b";` -> Membuat 2 objek string. | `StringBuilder sb = new StringBuilder("a"); sb.append("b");` -> Tetap 1 objek. |

-   **Kesimpulan:** Dalam metode `collectWords` yang melakukan banyak operasi `append` dan `delete` di dalam *loop* rekursif, menggunakan `StringBuilder` **jauh lebih cepat dan hemat memori** dibandingkan menggunakan `String` biasa.

---

## Slide 10: `Trie.java` - `generateCandidates` (Penjelasan Lengkap)

-   **Tujuan:** Mencoba memperbaiki kata salah eja dengan membuat semua variasi kata yang berjarak 1 "edit".

### Blok Inisialisasi
-   **Logika:** Kita menyiapkan 4 `List` terpisah untuk setiap tipe edit. Tujuannya adalah agar nanti kita bisa menggabungkannya sesuai urutan prioritas. Kata input juga langsung diubah ke huruf kecil.
    ```java
    List<String> replacements   = new ArrayList<>();
    List<String> transpositions = new ArrayList<>();
    List<String> deletions      = new ArrayList<>();
    List<String> insertions     = new ArrayList<>();
    String lowerWord = word.toLowerCase();
    ```

### Blok 1: Replacements (Ganti Huruf)
-   **Logika:** Mencoba mengganti setiap karakter di dalam kata dengan semua huruf dari 'a' sampai 'z'.
    ```java
    for (int i = 0; i < lowerWord.length(); i++) {
        char[] chars = lowerWord.toCharArray();
        for (char c = 'a'; c <= 'z'; c++) {
            chars[i] = c;
            String replaced = new String(chars);
            if (contains(replaced)) {
                replacements.add(replaced);
            }
        }
    }
    ```

### Blok 2: Transpositions (Tukar Posisi Huruf)
-   **Logika:** Menukar posisi dua karakter yang bersebelahan.
    ```java
    for (int i = 0; i < lowerWord.length() - 1; i++) {
        char[] chars = lowerWord.toCharArray();
        char tmp = chars[i];
        chars[i] = chars[i+1];
        chars[i+1] = tmp;
        String t = new String(chars);
        if (contains(t)) {
            transpositions.add(t);
        }
    }
    ```

### Blok 3: Deletions (Hapus Huruf)
-   **Logika:** Mencoba menghapus satu karakter dari setiap posisi.
    ```java
    for (int i = 0; i < lowerWord.length(); i++) {
        String deleted = lowerWord.substring(0, i) + lowerWord.substring(i + 1);
        if (contains(deleted)) {
            deletions.add(deleted);
        }
    }
    ```

### Blok 4: Insertions (Sisip Huruf)
-   **Logika:** Mencoba menyisipkan setiap huruf alfabet ke setiap kemungkinan posisi.
    ```java
    for (int i = 0; i <= lowerWord.length(); i++) {
        for (char c = 'a'; c <= 'z'; c++) {
            String inserted = new StringBuilder(lowerWord).insert(i, c).toString();
            if (contains(inserted)) {
                insertions.add(inserted);
            }
        }
    }
    ```

### Langkah Terakhir: Penggabungan
-   **Logika:** Semua kandidat dari 4 `List` di atas digabungkan menjadi satu `List` akhir sesuai urutan prioritas.
    ```java
    List<String> allCandidates = new ArrayList<>();
    allCandidates.addAll(insertions);
    allCandidates.addAll(deletions);      
    allCandidates.addAll(transpositions); 
    allCandidates.addAll(replacements);
    return allCandidates;
    ```
---

## Slide 11: `Trie.java` - Metode `loadDictionaryFromCSV`

### `public int loadDictionaryFromCSV(String csvPath)`
-   **Tujuan:** Memuat data kamus dari file `kbbi_v.csv` ke dalam struktur data Trie saat aplikasi pertama kali dijalankan.
-   **Cara Kerja:**
    1.  Membaca file CSV baris per baris.
    2.  Untuk setiap baris, ambil kata dari kolom pertama.
    3.  Panggil metode `insert(word)` untuk setiap kata yang valid untuk membangun Trie.
    4.  Menampilkan progress loading setiap 1000 kata.
-   **Kode:**
    ```java
    public int loadDictionaryFromCSV(String csvPath) {
        int wordCount = 0;
        try (BufferedReader br = new BufferedReader(new FileReader(csvPath))) {
            String line;
            while ((line = br.readLine()) != null) {
                int commaIndex = line.indexOf(',');
                String word = (commaIndex > 0) ? 
                    line.substring(0, commaIndex).trim() : 
                    line.trim();
                
                if (!word.isEmpty()) {
                    insert(word.toLowerCase());
                    wordCount++;
                }
            }
            System.out.println("Dictionary loading complete. Total words loaded: " + wordCount);
            return wordCount;
        } catch (IOException e) {
            System.err.println("Error loading dictionary from " + csvPath + ": " + e.getMessage());
            return wordCount;
        }
    }
    ```

---
---

## BAGIAN 2: BINARYSEARCHTREE.JAVA (MESIN PERSONALISASI)

---

## Slide 12: `BinarySearchTree.java` - Peran & Masalah Awal

-   **Peran:** Menyimpan frekuensi penggunaan setiap kata untuk personalisasi saran.
-   **Struktur:** Key-Value, `Key` adalah kata (`String`), `Value` adalah frekuensi (`Integer`).
-   **Masalah pada Kode Lama (dari Asdos):**
    -   Metode `insert` tidak bisa meng-update nilai dari `key` yang sudah ada. Ia hanya bisa menambah node baru.
    -   `find` menggunakan `==` untuk membandingkan `String`, yang tidak akurat.
    -   **Akibatnya:** Untuk update frekuensi, kita harus `delete` lalu `insert` lagi. Ini tidak efisien dan berisiko merusak struktur pohon.

---

## Slide 13: Penjelasan Kode `insertNode` (Baru vs. Lama)

**KODE LAMA**
```java
private BTNode<K,V> insertNode(BTNode<K,V> n, K k, V d) {
    if(n == null) return new BTNode<K,V>(k, d);
    
    // Hanya navigasi ke kiri atau kanan.
    // Tidak ada kondisi untuk key yang sama.
    if(k.compareTo(n.getKey()) < 0)
        n.setLlink(insertNode(n.getLlink(), k, d));
    else
        n.setRlink(insertNode(n.getRlink(), k, d));
    return n;
}
```

**KODE BARU YANG DIREFACTOR**
```java
private BTNode<K,V> insertNode(BTNode<K,V> node, K k, V data) {
    // Baris 1: Kondisi dasar rekursi. Jika kita menemukan
    //         posisi kosong, buat node baru di sini.
    if (node == null) return new BTNode<K,V>(k, data);

    // Baris 2: Bandingkan key baru (k) dengan key node saat ini.
    int cmp = k.compareTo(node.getKey());

    if (cmp < 0) // Jika lebih kecil, cari posisi di subtree kiri.
        node.setLlink(insertNode(node.getLlink(), k, data));
    else if (cmp > 0) // Jika lebih besar, cari posisi di subtree kanan.
        node.setRlink(insertNode(node.getRlink(), k, data));
    else // Jika SAMA (cmp == 0), ini adalah kuncinya!
         // Kita tidak buat node baru, tapi UPDATE datanya.
        node.setData(data);
    
    return node;
}
```

---

## Slide 14: Penjelasan Metode Publik - `insert` & `search`

### `public void insert(K key, V data)`
-   **Tujuan:** Ini adalah metode yang dipanggil dari luar (misal, dari `SpellCheckerMain`). Metode ini menyembunyikan kompleksitas rekursi dari pengguna.
-   **Cara Kerja:** Sangat sederhana. Ia hanya memulai proses rekursif dengan memanggil `insertNode`, dimulai dari `root` pohon.
    ```java
    public void insert(K key, V data) {
        root = insertNode(root, key, data);
    }
    ```

### `public V search(K key)`
-   **Tujuan:** Mencari sebuah `key` di dalam pohon dan mengembalikan `data` yang berasosiasi dengannya.
-   **Cara Kerja:**
    1.  Memanggil metode *private helper* `find(root, key)` untuk mencari node yang sesuai.
    2.  Jika `find` mengembalikan `null` (node tidak ditemukan), maka `search` juga mengembalikan `null`.
    3.  Jika `find` berhasil menemukan node, `search` akan mengembalikan data dari node tersebut.
    ```java
    public V search(K key) {
        BTNode<K,V> node = find(root, key);
        if (node == null) {
            return null;
        }
        return node.getData();
    }
    ```

---

## Slide 15: Penjelasan `private BTNode<K,V> find(...)`

Metode `find` adalah "mesin pencari" rekursif dari BST. Ini adalah implementasi klasik dari pencarian biner pada struktur pohon.

```java
private BTNode<K,V> find(BTNode<K,V> node, K k) {
    // KONDISI 1: Jika node saat ini null,
    // berarti kita sudah sampai ujung pohon dan tidak ketemu.
    if (node == null) {
        return null;
    }

    // Bandingkan key yang dicari (k) dengan key node saat ini.
    int cmp = node.getKey().compareTo(k);

    // KONDISI 2: Jika key SAMA (cmp == 0),
    // kita berhasil menemukan nodenya! Kembalikan node ini.
    if (cmp == 0) {
        return node;
    } 
    // KONDISI 3: Jika key node saat ini lebih kecil dari k,
    // berarti key yang kita cari ada di sebelah KANAN.
    // Lanjutkan pencarian di subtree kanan.
    else if (cmp < 0) {
        return find(node.getRlink(), k);
    } 
    // KONDISI 4: Jika key node saat ini lebih besar dari k,
    // berarti key yang kita cari ada di sebelah KIRI.
    // Lanjutkan pencarian di subtree kiri.
    else {
        return find(node.getLlink(), k);
    }
}
```

---

## Slide 16: Mengapa Refactoring BST Penting?

1.  **Logika Update Menjadi Sederhana:**
    -   **Sebelumnya:** Proses update frekuensi di `SpellCheckerMain` harus: `search` -> `delete` -> `insert`.
    -   **Sesudah:** Cukup `search` -> `insert`. Metode `insert` yang baru sudah cukup "pintar" untuk menangani kasus update data.
2.  **Menjaga Integritas Struktur Pohon:**
    -   Operasi `delete` pada BST adalah operasi yang kompleks dan bisa menyebabkan rotasi atau perubahan besar pada struktur pohon. Dengan menghilangkan `delete` dari proses update, kita membuat prosesnya lebih aman dan *predictable*.
3.  **Perbaikan `find`:** Kode lama menggunakan `key == node.getKey()` yang membandingkan referensi objek, bukan nilainya. Kode baru menggunakan `compareTo` yang membandingkan isi `String` dengan benar, memastikan pencarian akurat.

---
---

## BAGIAN 3: SPELLCHECKERMAIN.JAVA (ORKESTRATOR UTAMA)

---

## Slide 17: `SpellCheckerMain.java` - Alur Utama & Metode `main`

-   **Peran:** Bertindak sebagai "otak" atau orkestrator aplikasi. Ia mengelola alur program, menerima input user, dan menggunakan `Trie` serta `BST` untuk menjalankan tugasnya.
-   **Langkah-langkah di `main`:**
    1.  **Inisialisasi:** Membuat objek `Trie` untuk kamus dan `BinarySearchTree` untuk frekuensi.
    2.  **Memuat Kamus:** Mengisi `Trie` dengan puluhan ribu kata dari file `kbbi_v.csv`. Ini adalah proses satu kali saat startup.
    3.  **Loop Interaktif:** Menjalankan `while(true)` loop yang menampilkan menu utama ("Periksa Teks" atau "Keluar").
    4.  **Pemrosesan Input & Delegasi:** Saat pengguna memilih untuk memeriksa teks, program meminta input, memecahnya menjadi kata-kata, lalu memanggil `processWord()` untuk setiap kata.
    5.  **Output:** Setelah semua kata diproses, program menampilkan teks asli dan teks yang sudah dikoreksi secara berdampingan.
-   **Kode `main`:**
    ```java
    public static void main(String[] args) {
        // 1. Startup & Inisialisasi
        dictionary = new Trie();
        freqBst = new BinarySearchTree<>();
        
        // 1.a Load dictionary ke Trie
        System.out.println("Loading dictionary...");
        int wordsLoaded = dictionary.loadDictionaryFromCSV("src/com/kbbi_v.csv");
        System.out.println("Successfully loaded " + wordsLoaded + " words into the dictionary.");

        // 1.b BST sudah dibuat kosong di atas
        while (true) {
            System.out.println("\n=== Spell Checker ===");
            System.out.println("1. Check text");
            System.out.println("2. Exit");
            System.out.print("Choose option: ");
            int choice = scanner.nextInt();
            scanner.nextLine();

            if (choice == 2) break;
            if (choice == 1) {
                System.out.println("Enter text to check:");
                String text = scanner.nextLine();
                
                String[] words = text.toLowerCase().split("\\s+");
                String[] correctedWords = new String[words.length];
                
                for (int i = 0; i < words.length; i++) {
                    correctedWords[i] = processWord(words[i]);
                }
                
                System.out.println("\nOriginal text: " + text);
                System.out.println("Corrected text: " + String.join(" ", correctedWords));
            }
        }
        scanner.close();
    }
    ```

---

## Slide 18: `SpellCheckerMain.java` - Alur `processWord` (Bagian 1)

-   **Tujuan:** Menerima satu kata, memutuskan apakah kata itu valid atau salah eja, dan mengembalikan versi yang benar.
-   **Langkah 1: Filter & Cek Validitas**
    1.  **Filter Non-Huruf:** Kata yang mengandung karakter selain `a-z` (misal: "123", "halo!") akan langsung dikembalikan tanpa dicek.
    2.  **Cek Kamus:** Jika kata valid (ada di `Trie`), program akan mencetak status "Valid word", meng-update frekuensi kata tersebut di `BST`, dan mengembalikan kata itu sendiri.

- **Kode:**
    ```java
    private static String processWord(String word) {
        // Cek apakah token HANYA terdiri dari huruf a-z.
        if (!word.matches("[a-z]+")) {
            return word;
        }

        System.out.print("Checking '" + word + "': ");
        
        // Untuk kata valid
        if (dictionary.contains(word)) {
            System.out.println("Valid word");
            // Update frekuensi di BST
            updateFrequency(word);
            return word; // Kembalikan kata yang valid
        } 
        else {
            // ... Logika untuk kata salah eja (lanjut ke slide berikutnya)
            return handleMisspelledWord(word); // Panggilan ke helper, dijelaskan berikutnya
        }
    }
    ```
    *(Catatan: Kode di atas sedikit disederhanakan untuk presentasi dengan memindahkan logika `else` ke helper `handleMisspelledWord`)*

---

## Slide 19: `SpellCheckerMain.java` - Alur `processWord` (Bagian 2: Koreksi)

-   **Langkah 2: Proses Koreksi Kata Salah Eja**
    1.  **Generate & Sort:** Jika kata tidak ditemukan, `generateCandidates()` dipanggil. Hasilnya langsung di-sortir berdasarkan frekuensi dari `BST` (`getFrequency`).
    2.  **Tampilkan Saran:** Tampilkan maksimal 3 saran teratas ke pengguna, lengkap dengan informasi frekuensi (`used X times`). Jika tidak ada saran, kembalikan kata asli.
    3.  **Input Pengguna:** Minta pengguna memilih koreksi atau skip.
    4.  **Update & Kembalikan:** Jika pengguna memilih koreksi, frekuensi kata yang **terpilih** akan di-update di `BST`, dan kata tersebut dikembalikan. Jika tidak, kata asli dikembalikan.

- **Kode (Logika di dalam blok `else` atau helper `handleMisspelledWord`):**
    ```java
    // Blok 'else' dari slide sebelumnya
    System.out.println("Misspelled word");
    
    List<String> candidates = dictionary.generateCandidates(word);
    
    // Sortir berdasarkan frekuensi, descending
    candidates.sort((a, b) -> getFrequency(b) - getFrequency(a));

    if (candidates.isEmpty()) {
        System.out.println("No suggestions found.");
        return word;
    }

    System.out.println("Suggestions:");
    int numSuggestions = Math.min(3, candidates.size());
    for (int i = 0; i < numSuggestions; i++) {
        String candidate = candidates.get(i);
        int freq = getFrequency(candidate);
        System.out.printf("%d. %s (used %d times)\n", i+1, candidate, freq);
    }

    System.out.print("Choose correction (1-" + numSuggestions + ") or 0 to skip: ");
    int choice = scanner.nextInt();
    scanner.nextLine();

    if (choice > 0 && choice <= numSuggestions) {
        String selectedWord = candidates.get(choice-1);
        updateFrequency(selectedWord);
        System.out.println("Corrected to: " + selectedWord);
        return selectedWord;
    }
    
    return word; // User memilih untuk skip
    ```

---

## Slide 20: `SpellCheckerMain.java` - Helpers untuk Personalisasi

-   **Tujuan:** Dua metode ini (`getFrequency` dan `updateFrequency`) membungkus interaksi dengan `BinarySearchTree` agar kode di `processWord` lebih bersih dan aman.

-   **`private static int getFrequency(String word)`**
    -   **Fungsi:** Mengambil data frekuensi sebuah kata dari BST.
    -   **Penanganan Null:** Sangat penting. Jika `freqBst.search(word)` mengembalikan `null` (artinya kata belum pernah digunakan), metode ini mengembalikan `0`, bukan `null`. Ini mencegah `NullPointerException` saat menyortir kandidat.
    ```java
    private static int getFrequency(String word) {
        Integer freq = freqBst.search(word);
        return (freq != null) ? freq : 0;
    }
    ```

-   **`private static void updateFrequency(String word)`**
    -   **Fungsi:** Meng-update (menambah atau menyisipkan) data frekuensi sebuah kata.
    -   **Koneksi ke Refactoring BST:** Metode ini mengandalkan `insert` dari `BST` yang sudah di-refactor. Ia mencari frekuensi saat ini, lalu memanggil `freqBst.insert(word, newFrequency)`. Karena `insert` kita bisa menangani *update*, kode ini berfungsi untuk kata yang baru maupun yang sudah ada.
    ```java
    private static void updateFrequency(String word) {
        Integer currentFreq = freqBst.search(word);
        if (currentFreq == null) {
            // Kata baru, masukkan dengan frekuensi 1
            freqBst.insert(word, 1);
        } else {
            // Kata sudah ada, increment frekuensi
            freqBst.insert(word, currentFreq + 1);
        }
    }
    ```
-   **Kesimpulan:** Dua helper ini adalah jantung dari fitur personalisasi, membuat interaksi dengan BST menjadi sederhana dan kuat.

---

## Slide 21: Demo Aplikasi & Kesimpulan

*(Sebelumnya Slide 20, berisi rencana demo dan kesimpulan proyek)* 

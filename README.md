# Cracking bcrypt Hashes with Hashcat

## What this is

This repository documents a real exercise I did to crack bcrypt password hashes from a leaked SQL database. I extracted the hashes and used Hashcat to recover the original passwords.

## The problem

I had a SQL dump file containing user passwords stored as bcrypt hashes. I needed to crack them to understand the password strength and patterns.

### The hash format
- **Algorithm**: bcrypt
- **Identifier**: `$2b$`
- **Cost factor**: 10
- **Example**: `$2b$10$wE7FqSdqnoonSzCL06mLJcoqmK6PNBJm2ZK38fg3ET12VrfqODNL`

## What I tried first (that failed)

I ran Hashcat inside a Kali Linux VM on my Mac.

- **Hashcat mode**: `-m 3200` (bcrypt)
- **Attack mode**: `-a 0` (dictionary attack)
- **Wordlist**: rockyou.txt (14 million passwords)

**Result**: Hashcat estimated **48 days** to complete. The VM was using CPU only, no GPU acceleration.

## What actually worked

I moved the hashes to my Mac host machine and ran Hashcat natively on macOS with GPU acceleration.

### Steps I followed

**1. Extract hashes from the SQL file**
```bash
grep -oE '\$2[aby]\$[0-9]{2}\$[./A-Za-z0-9]{53}' database.sql > hashes.txt
```

**2. Transfer hashes from Kali VM to Mac**
I started a simple HTTP server in Kali:
```bash
python3 -m http.server 8000
```

**3. Run Hashcat on macOS**
```bash
hashcat -m 3200 -a 0 hashes.txt rockyou.txt -o cracked.txt -O -w 3
```

**4. View cracked passwords**
```bash
hashcat -m 3200 --show hashes.txt
```

## Key lesson
When using hashcat to crack bcrypt hashes (Mode 3200), the environment significantly impacts speed. 
Virtual machines (VMs) often lack direct access to hardware acceleration, while native macOS systems can leverage powerful Metal (GPU) acceleration.
Note on Bcrypt: Even on fast hardware, bcrypt is designed to be slow. 

## Files in this repo

    scripts/extract_hashes.sh - Extract bcrypt hashes from SQL dump

    scripts/crack.sh - Run Hashcat with optimal settings

    samples/hash_example.txt - Example bcrypt hash

## Commands reference
### Extract hashes
./scripts/extract_hashes.sh database.sql hashes.txt

### Crack with Hashcat
./scripts/crack.sh hashes.txt /path/to/rockyou.txt

### Check results
hashcat -m 3200 --show hashes.txt


## Tools used

    Hashcat 6.x - GPU-accelerated password recovery

    rockyou.txt - Wordlist (14 million passwords)

    Kali Linux - Initial extraction (VM)

    macOS - Final cracking (host)

## Disclaimer
This was done on a leaked database for educational purposes. Always get proper authorization before attempting password recovery.

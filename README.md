# HoneyPot

# Step 1: Core Setup


Tools & Base System
OS: Use Ubuntu Server 22.04 LTS (minimal install).

Honeypot Software: Modify Cowrie (open-source SSH honeypot) as the base.


git clone https://github.com/cowrie/cowrie
cd cowrie
virtualenv --python=python3 cowrie-env
source cowrie-env/bin/activate
pip install -r requirements.txt

# Port Configuration
# Redirect SSH traffic (port 22) to Cowrie’s port (2222):


iptables -t nat -A PREROUTING -p tcp --dport 22 -j REDIRECT --to-port 2222


# Step 2: Add-Ons

1. Blockchain Immutable Logging
Concept: Log all attacker activity to a private blockchain (Ethereum-based) to ensure logs are tamper-proof.

# Implementation:

I used Ganache (local Ethereum blockchain) for testing.

# Python script (log_to_blockchain.py) to send Cowrie logs (commands, IPs, timestamps) as blockchain transactions.

from web3 import Web3
w3 = Web3(Web3.HTTPProvider('http://localhost:7545'))
account = w3.eth.accounts[0]
tx = w3.eth.send_transaction({
    'from': account,
    'to': account,
    'value': 0,
    'data': '0x' + log_data.encode().hex()
})

2. AI-Driven Dynamic Environment
Concept: Used a pre-trained ML model to classify attackers (script kiddie, bot, advanced) and alter the honeypot’s responses.

Implementation:

Training a model (TensorFlow/PyTorch) on existing Cowrie logs to predict threat levels.

Integrating with Cowrie’s fs.pickle (fake filesystem) to dynamically generate fake files/directories based on the attacker’s behavior.

Example: If the attacker is a bot, serve a fake /etc/passwd with 10,000 users to slow them down.

3. Time Dilation & Fake Vulnerabilities
Concept: Introduced random delays or fake "vulnerabilities" when attackers trigger specific commands (e.g., sudo su, wget).

Implementation:

Modified Cowrie’s shell.py to add delays using time.sleep().

Simulate a shell crash on rm -rf / and log the attacker’s reaction.

4. Dockerized Ephemeral Sessions
Concept: Spined up a fresh Docker container for each attacker session to prevent cross-contamination.

Implementation:


docker run -d --name honeypot_session_$RANDOM -p 2222:2222 cowrie_image
Use docker-compose to auto-destroy containers after 24 hours.

5. Gamified "HoneyTokens"
Concept: Hide fake API keys, passwords, or flags in the honeypot. When attackers use them, trigger alerts.

Implementation:

Place fake AWS keys in ~/.aws/credentials. Monitor AWS logs for usage.

Used Canarytokens (free service) for automated alerts.

# Step 3: Telemetry & Analytics

Real-Time Dashboard
Used Grafana + Elasticsearch to visualize geolocation of attackers.

Command frequency (heatmap).

Threat level classifications from the AI model.

Behavioral Fingerprinting
Track unique attacker behaviors:

Typing speed (time between commands).

Common typos/mistakes.

Preferred exploitation tools (e.g., hydra, nmap).

# Step 4: Deployment & Defense

Isolation: Deploy the honeypot on a separate VLAN or cloud instance (AWS/Azure/DigitalOcean).

Camouflage: Rename the honeypot binary to sshd and use common systemd service names.

Alerting: Integrate with Shodan to monitor for your honeypot’s IP being indexed.



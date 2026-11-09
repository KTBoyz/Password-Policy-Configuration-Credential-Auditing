Repository Structure
.
├── README.md
├── images/
│   ├── Phase1/
│   ├── Phase2/
│   └── Phase3/
└── report/
    └── Password_Policy_Configuration_and_Credential_Auditing.pdf

Password Policy Configuration and Credential Auditing
Introduction

Passwords remain one of the most commonly targeted authentication mechanisms in modern computing environments. Weak, predictable, or poorly managed passwords significantly increase the risk of unauthorized access, credential theft, and privilege escalation. To mitigate these risks, modern operating systems implement password policies that enforce complexity requirements and secure password storage mechanisms.

This project focuses on strengthening local account security in Kali Linux through the configuration of Password Authentication Modules (PAM) and subsequent credential auditing. The assessment begins by enforcing password complexity requirements using the libpam-pwquality module and modifying system-wide password quality settings. After implementing these controls, a series of credential-auditing activities are performed to evaluate the effectiveness of the configured password policy.

The project demonstrates both defensive and auditing perspectives of password security. Defensive controls are implemented through PAM-based password policies, while auditing is carried out through password hash extraction, offline credential analysis, and password auditing tools such as John the Ripper and Hashcat. The overall objective is to understand how password policies influence password strength, storage security, and resistance to common password-cracking techniques.

Objectives
-Configure and validate password complexity requirements using PAM.
-Install and configure the libpam-pwquality password quality module.
-Enforce minimum password length and character diversity requirements.
-Restrict the use of passwords based on usernames or predictable patterns.
-Evaluate the effectiveness of configured password policies.
-Extract password hashes from the Linux authentication database.
-Prepare password hashes for offline auditing using the unshadow utility.
-Perform credential auditing using John the Ripper.
-Analyze password candidate generation strategies including username-based and rule-based attacks.
-Investigate password auditing limitations associated with modern hashing algorithms such as yescrypt.
-Assess the effectiveness of password policies against offline password-cracking techniques.
-Document security findings and recommendations for improving credential security.

Tools and Technologies
-Kali Linux
-PAM (Pluggable Authentication Modules)
-libpam-pwquality
-chage
-John the Ripper
-Hashcat
-RockYou Wordlist
-Openwall John Source Repository

Skills Demonstrated
-Linux System Administration
-PAM Configuration
-Password Policy Enforcement
-Account Lifecycle Management
-Password Auditing
-Linux Authentication Architecture
-Hash Analysis
-John the Ripper
-Hashcat
-Security Assessment
-Security Documentation

Phase 1 - Configuring Password Complexity via PAM

Sub step 1: Installation and verification of libpam-pwquality

Commands Executed:

Installation...
sudo apt-get install -y --fix-missing libpam-pwquality

Verification...
dpkg -l | grep libpam-pwquality

Results:

<img width="1346" height="457" alt="1 1_installing_verify_pam_phase1" src="https://github.com/user-attachments/assets/c74f71fc-79f3-4d3a-b1d5-41eb3aefaf3e" />

Observation

The libpam-pwquality module was successfully installed and verified on the Kali Linux system. The package provides PAM functionality for enforcing password quality and complexity requirements.

Analysis

The installation of libpam-pwquality enables the operating system to evaluate password strength during password creation and modification events. This module serves as the primary component responsible for enforcing password quality controls throughout the project.

Sub step 2: Accessing the Password Quality Configuration File and Password Complexity Configuration

Commands Executed:

Accessing Configuration file...
sudo nano /etc/security/pwquality.conf

Results:

<img width="1919" height="782" alt="1 3_pwquality_changes_phase1" src="https://github.com/user-attachments/assets/02c12cd8-836c-4d1d-9b1e-aafc077a7e3b" />

Observation

The system password quality configuration file was opened with administrative privileges for policy customization.

The password policy was configured to require:

Minimum length of 12 characters.
At least one numeric character.
At least one uppercase character.
At least one lowercase character.
At least one special character.
Restricted character repetition.

Modified Parameters

minlen = 12
dcredit = -1
ucredit = -1
lcredit = -1
ocredit = -1
maxrepeat = 3

Analysis

The pwquality.conf file provides centralized control over password complexity requirements. By modifying this file, security administrators can implement organization-wide password policies without modifying individual user accounts.

These controls significantly increase password entropy and reduce susceptibility to dictionary-based and rule-based password attacks. By requiring multiple character classes and longer password lengths, the policy forces users to create more complex credentials that are less likely to appear in password dictionaries or predictable attack patterns.

Sub step 3: Integrating Password Quality Controls into PAM

Commands Executed:

Changing Configurations...
sudo nano /etc/pam.d/common-password

Configurations Added:
password requisite pam_pwquality.so retry=3 minlen=12 \
ucredit=-1 lcredit=-1 dcredit=-1 ocredit=-1 enforce_for_root

Results:

<img width="1902" height="720" alt="1 4_changing_pam_policies" src="https://github.com/user-attachments/assets/b7ca3109-eb7b-42b9-a4f4-21272602e20b" />

Observation

The PAM password stack was modified to invoke the pam_pwquality module whenever a user attempts to create or change a password. The configuration enforces password-quality requirements, including minimum length, character diversity, and validation for privileged accounts.

The following controls were configured:

minlen=12 → Minimum password length of 12 characters.
ucredit=-1 → At least one uppercase character required.
lcredit=-1 → At least one lowercase character required.
dcredit=-1 → At least one numeric character required.
ocredit=-1 → At least one special character required.
retry=3 → Users are given three attempts to create a compliant password.
enforce_for_root → Password complexity requirements are enforced for the root account as well.

Analysis

While pwquality.conf defines the organization's password-quality rules, those rules are not enforced until the PAM authentication framework is instructed to use the pam_pwquality module. This step integrates the configured password policy directly into the authentication process.

By placing pam_pwquality.so in the password stack before password updates are committed, the system validates every proposed password against the configured security requirements. Any password failing to satisfy the minimum length, character-class requirements, repetition limits, or similarity checks is rejected before being stored.

This configuration establishes the primary defensive control of the project, ensuring that weak, predictable, or username-based passwords are blocked at creation time rather than being detected after deployment through credential auditing.

Sub step 4: Validating Password Policy Enforcement

Commands Executed:

Assigning Password...
sudo passwd policytest

Results:

<img width="622" height="209" alt="1 5_testing_pam_policy_phase1" src="https://github.com/user-attachments/assets/e1f493cc-eb77-4757-883a-6a0d5d2498c7" />

Observation

The configured PAM password policy was tested by attempting to assign passwords that violated the defined complexity requirements.

During testing, the system rejected multiple password attempts and returned the following validation errors:

BAD PASSWORD: The password contains less than 1 digits
BAD PASSWORD: The password contains less than 1 uppercase letters
BAD PASSWORD: The password is shorter than 12 characters
passwd: Have exhausted maximum number of retries for service
passwd: password unchanged

The password update operation was ultimately denied after the maximum number of permitted retry attempts was exceeded.

Analysis

This test confirms that the previously configured libpam-pwquality and PAM settings were successfully enforced by the operating system. The password validation engine evaluated every submitted password against the configured security controls and rejected those that failed to satisfy the established requirements.

The observed failures directly correspond to the configured policy:

minlen = 12
ucredit = -1
dcredit = -1
lcredit = -1
ocredit = -1

The rejection messages demonstrate that the system actively enforces:

Minimum password length requirements.
Mandatory numeric characters.
Mandatory uppercase characters.
Character diversity rules.
Limited retry attempts.

By preventing weak passwords from being stored in the authentication database, the system reduces exposure to password-guessing, dictionary-based, and rule-based credential attacks.

Conclusion

The Password Authentication Module (PAM) framework was successfully configured to enforce password complexity requirements using the libpam-pwquality module. Password quality controls were implemented through modifications to both pwquality.conf and the PAM password stack.

Testing confirmed that passwords failing to meet the configured requirements were automatically rejected, demonstrating successful policy enforcement. The implemented controls establish a stronger first line of defense by ensuring that weak, predictable, or easily guessable passwords cannot be used during account creation or password modification processes.

The successful enforcement of password complexity requirements provides a secure foundation for the credential auditing activities performed in the subsequent phase of this project.

Phase 2 – Account Lifecycle Management
Overview

While Phase 1 focused on enforcing password complexity during password creation, Phase 2 focuses on managing the lifecycle of user credentials after deployment. Password lifecycle management is a critical security control that ensures credentials are changed periodically, prevents indefinite password reuse, and provides mechanisms for account inactivity handling and password expiration warnings.

The objective of this phase is to configure password-aging policies for a test account and verify that the configured controls are enforced during authentication.

Sub Step 1: Inspecting Existing Password Lifecycle Configuration

Command Executed:

Password Lifecycle Inspection...
sudo chage -l policytest

Results:

<img width="671" height="211" alt="2 1_policytest_inspection" src="https://github.com/user-attachments/assets/4d728555-4791-4c68-acd9-041603506111" />

Observation

The current password-aging configuration for the policytest account was displayed.

The initial configuration showed:

Password expires : Aug 19, 2300
Password inactive : never
Account expires : never
Minimum days between changes: 0
Maximum days between changes: 99999
Warning days : 7

Analysis

The account was effectively operating without meaningful password-aging controls. Passwords could remain unchanged for extremely long periods, increasing the risk that compromised credentials might remain valid indefinitely.

The inspection phase established the baseline configuration before implementing lifecycle management policies.

Sub Step 2: Configuring Password Aging Policies

Commands Executed:

Configure Password Aging...
sudo chage -M 90 -m 7 -W 7 -I 30 policytest

Force Password Change on Next Login...
sudo chage -d 0 policytest

Results:

<img width="530" height="115" alt="2 2_pwconfig_pwreset" src="https://github.com/user-attachments/assets/d801fb06-9a8c-48f0-8cc2-6d8881f8b0b9" />

Observation

Password lifecycle controls were applied to the policytest account.

The configured parameters included:

-M 90 → Password expires after 90 days
-m 7 → Minimum 7 days before another password change
-W 7 → Warn user 7 days before password expiration
-I 30 → Disable account after 30 days of inactivity
-d 0 → Force password change at next login

Analysis

These controls reduce the risk associated with long-lived credentials. Regular password rotation limits the usefulness of compromised passwords, while warning periods help users update credentials proactively. The forced password reset mechanism ensures that administrators can immediately invalidate previously assigned credentials and require the user to establish a new password.

Sub Step 3: Verifying Lifecycle Policy Configuration

Command Executed:

Verifying Configuration...
sudo chage -l policytest
 
Results:

<img width="782" height="192" alt="2 3_pwchange_verification" src="https://github.com/user-attachments/assets/28ebf362-b501-41ef-b3b1-3dcd81682a63" />

Observation

The updated password-aging policy was successfully applied.

Output confirmed:

Last password change : password must be changed
Password expires : password must be changed
Password inactive : password must be changed
Minimum days between password change: 7
Maximum days between password change: 90
Number of warning days : 7

Analysis

The verification confirms that the account lifecycle configuration was successfully implemented. The system now maintains a controlled password management process, requiring periodic credential updates and enforcing password expiration policies.

Sub Step 4: Live Verification of Forced Password Change

Command Executed:

Live Verification...
su - policytest

Results:

<img width="727" height="177" alt="2 4_live_passwd_change_verification" src="https://github.com/user-attachments/assets/55f25a3a-ed05-44a7-80ec-7d679e7fe0ee" />

Observation

Upon attempting to log in as policytest, the system immediately displayed:

You are required to change your password immediately (administrator enforced).

The user was then prompted to:

Current password:
New password:
Retype new password:

before being granted normal access.

Analysis

This live test confirms that the lifecycle controls are not merely stored in configuration files but are actively enforced during authentication. The system successfully recognized that the password was marked for mandatory reset and prevented normal account usage until a new password was provided.

This behavior demonstrates the effectiveness of password-aging policies in enforcing credential renewal and reducing exposure to stale or administrator-assigned passwords.

Phase 2 Conclusion

The account lifecycle management phase successfully implemented and validated password-aging controls using the Linux chage utility. Password expiration, minimum password age, warning periods, inactivity controls, and forced password changes were configured and verified for the policytest account.

Testing confirmed that the operating system actively enforced the configured policies during authentication, requiring immediate password replacement when administrative reset conditions were triggered.

Together with the PAM-based complexity controls implemented in Phase 1, these lifecycle-management mechanisms establish a stronger credential-security framework by controlling not only password strength but also password longevity and renewal behavior.

Phase 3 – Credential Auditing and Security Assessment

Section A – Credential Auditing Using John the Ripper

Overview

The objective of this phase is to evaluate the effectiveness of the password security controls implemented in the previous phases. This assessment simulates an offline password-auditing scenario in which an attacker obtains access to local authentication databases and attempts to recover plaintext credentials through password-cracking techniques.

John the Ripper was selected as the primary auditing tool due to its support for modern Linux password formats and its ability to perform Single Crack, Rule-Based, and Dictionary-Based attacks against extracted password hashes.

Supporting Step A.1 – John the Ripper Environment Preparation

Commands Executed

Updating John the Ripper...
sudo apt update

Installation Phase...
sudo apt install -y git build-essential libssl-dev zlib1g-dev yasm pkg-config libgmp-dev libcap-dev libbz2-dev

Results

<img width="1226" height="292" alt="z1_upgrading_john" src="https://github.com/user-attachments/assets/d6d35074-9fdb-4725-bbe8-9bb850669fcd" />

Observation

The operating system repositories were updated and the required development dependencies were installed successfully.

Analysis

Password-auditing tools rely on cryptographic and system libraries to support modern password hashing formats. Installing the required dependencies ensures the auditing platform can process contemporary Linux password hashes and supports future upgrades of auditing utilities.

Supporting Step A.2 – Obtaining the Latest John the Ripper Source

Command Executed

Obtaining Latest Source...
git clone https://github.com/openwall/john.git /tmp/john-src

Results

<img width="1226" height="206" alt="z2_cloning_github_john" src="https://github.com/user-attachments/assets/b24abcb2-8c78-4a52-967d-717ac805f615" />

Observation

The latest John the Ripper source repository was successfully cloned from the official Openwall project.

Analysis

This step was performed to investigate support for modern Linux password-hashing algorithms and to verify the capabilities of newer releases beyond the system-installed version.

Supporting Step A.3 – Investigating Yescrypt Compatibility

Commands Executed

Listing Formats...
sudo john --list=formats | grep -i yescrypt

Searching Yescrypt Format...
sudo john --list=formats | grep -Ei "yescrypt|crypt"

Results

<img width="725" height="345" alt="z3_format_listing_crypt" src="https://github.com/user-attachments/assets/9deddaca-549c-455c-abef-03a03dbca42c" />

Observation

A dedicated yescrypt format was not listed directly. However, the generic:

crypt

format was available.

Analysis

Inspection of the target account later revealed that Kali Linux stored passwords using the yescrypt algorithm, identified by the:

$y$

prefix.

Although John the Ripper did not expose yescrypt as a dedicated format, the generic:

--format=crypt

mode successfully processed the hashes, enabling credential auditing to proceed.

Sub Step 1 – Preparing Credential Databases

Commands Executed

Unshadowing /etc/passwd and /etc/shadow...
sudo unshadow /etc/passwd /etc/shadow > combined_hash.txt

Verification...
cat combined_hash.txt

Results

<img width="881" height="742" alt="3 1 1_unshadow_passwd_shadow_cat_verify" src="https://github.com/user-attachments/assets/837df1d8-27c5-4e6c-bcdb-d5bb75fed709" />

Observation

The contents of /etc/passwd and /etc/shadow were merged into a single auditing file called:

combined_hash.txt

Analysis

The resulting file contained both account metadata and password hashes required by John the Ripper for offline password analysis.

Sub Step 2 – Verifying Target Account Information

Commands Executed

Verifying Password Hash...
sudo grep "policytest" /etc/shadow

Verifying Account Information...
sudo grep "policytest" /etc/passwd

Results

<img width="824" height="124" alt="3 1 2_verification_test_account_shadow_passwd" src="https://github.com/user-attachments/assets/1b582212-fab0-4e89-b449-b1dc48795788" />

Observation

The policytest account was successfully located within both authentication databases.

The shadow entry contained the prefix:

$y$

which identifies the password-hashing algorithm.

Analysis

The password was stored using yescrypt, a modern password-hashing algorithm designed to resist password-cracking attacks through computational and memory-intensive processing.

Sub Step 3 – Preparing the Password Dictionary

Command Executed

Unzipping RockYou...
sudo gzip -d /usr/share/wordlists/rockyou.txt.gz

Results

<img width="530" height="55" alt="3 1 3_unzipping_rockyou" src="https://github.com/user-attachments/assets/f36a01f7-56e1-415f-9138-f9e10ef0dfcf" />

Observation

The RockYou password dictionary was successfully decompressed.

Analysis

The RockYou dataset contains a large collection of real-world passwords gathered from public credential breaches and is widely used for password auditing and password-strength assessments.

Sub Step 4 – Launching Credential Auditing

Command Executed

Credential Auditing...
sudo john --format=crypt combined_hash.txt

Results

<img width="1044" height="408" alt="3 1 4_credential_auditing" src="https://github.com/user-attachments/assets/24f1da32-065a-4d0d-a8cd-dd3d8edb8734" />

Observation

John the Ripper successfully loaded:

3 password hashes

and initiated:

Single Crack Mode

using:

rules:Single

Examples of generated candidates included:

weaktest
weaktest5
policytest?
kashyap5

Analysis

Single Crack Mode exploits information already present within account metadata, generating password candidates based on usernames and predictable user behavior.

This attack phase targets users who create passwords derived from account names or personal identifiers.

Sub Step 5 – Monitoring Password Candidate Generation

Results

<img width="1229" height="712" alt="z4_weaktest_john_crack" src="https://github.com/user-attachments/assets/164997b7-e658-473e-948e-47ae77736a22" />

Observation

The generated password candidates were heavily influenced by known usernames and account information.

Examples included:

weaktest5
weaktest80
weaktest90
weaktest123
policytest?
 
Analysis

The generated candidates clearly demonstrate how attackers exploit personal information when constructing password guesses.

This observation validates the importance of the PAM controls implemented in Phase 1, particularly controls preventing passwords derived from usernames.

Sub Step 6 – Monitoring Dictionary-Based Progress

Results

<img width="1044" height="408" alt="3 1 5_progress_monitor_eta" src="https://github.com/user-attachments/assets/e1f629cf-409f-4448-9f10-02bc420bae66" />

Observation

After completing Single Crack Mode, John automatically transitioned to:

Wordlist Mode

using:

/ usr/share/john/password.lst

Generated candidates included:

Hammer
Patrick
Thunderbird
Garfield
Alice1

while displaying:

Progress percentage
Candidate generation speed
Estimated completion time

Analysis

Dictionary-based attacks attempt commonly selected passwords and real-world password patterns collected from historical security breaches.

This phase represents how password attacks are commonly performed in real-world auditing engagements.

Sub Step 7 – Reviewing Audit Results

Command Executed

Derived Hash Result...
sudo john --show combined_hash.txt

Results

<img width="319" height="75" alt="3 1 6_john_output" src="https://github.com/user-attachments/assets/2cc69cca-bae4-45df-a7e4-b36c97d8f7f2" />

Observation

Output:

0 password hashes cracked, 0 left

No account passwords were successfully recovered.

Analysis

Despite testing username-derived candidates, rule-based transformations, and dictionary entries, John the Ripper failed to recover any passwords.

This result indicates that the password-complexity controls configured during Phase 1 successfully prevented the use of weak, predictable, or commonly selected passwords.

The auditing process therefore validated the effectiveness of the implemented password policy.

Section A Conclusion

The credential-auditing assessment successfully demonstrated the complete offline password-auditing workflow using John the Ripper. Authentication databases were extracted and prepared, password hashes were analyzed, and multiple attack methodologies were applied, including Single Crack and Dictionary-Based attacks.

The audit revealed that username-derived password guessing remains a common attack strategy and highlighted the importance of preventing predictable password patterns during account creation. Despite extensive candidate generation and dictionary testing, no passwords were recovered, demonstrating the effectiveness of the PAM-based password policies and password-lifecycle controls implemented in earlier phases.

The results confirm that strong password policies significantly improve resistance against common offline password-cracking techniques and serve as an effective first line of defense against credential compromise.

Section B – Credential Auditing Using Hashcat

Overview

To complement the John the Ripper assessment, Hashcat was evaluated as an alternative credential-auditing platform. The objective of this phase was to determine whether the extracted Linux password hashes could be processed using Hashcat's password-cracking engine and to investigate compatibility between the password-storage mechanisms used by modern Kali Linux systems and Hashcat attack modes.

Unlike the John the Ripper assessment, this phase focused heavily on hash identification, hash-format compatibility, and attack-mode selection.

Sub Step 1 – Rule-Based Candidate Generation Review

Command Executed

Checking Out John Rule Base...
john --wordlist=/usr/share/john/password.lst --rules --stdout | head -20

Results

<img width="638" height="393" alt="3 2 1_john_rule_set" src="https://github.com/user-attachments/assets/a465875f-44d0-4220-be75-8661e9e91e09" />

Observation

John the Ripper generated password candidates by applying rule-based transformations to entries contained within the supplied password dictionary.

Examples included:

123456
12345
password
password1
1234567890
abc123
computer
tigger
qwerty
money
secret
internet

The generated output demonstrated how rule engines expand password dictionaries to produce additional password candidates commonly selected by users.

Analysis

The --rules parameter extends dictionary-based attacks by automatically modifying existing dictionary entries.

Examples of rule-based transformations include:

Character substitution
Appending numbers
Capitalization changes
Symbol insertion
Character duplication
Pattern-based modifications

Rather than testing only known passwords, the rule engine generates realistic password variations that closely resemble the passwords commonly chosen by users.

This process illustrates how password-auditing tools simulate attacker behavior by targeting predictable password-selection habits.

Sub Step 2 – Hashcat Attack Attempt and Hash Compatibility Assessment

Command Executed

Loading Hash...
hashcat -m 1800 -a 0 policy_hash.txt /usr/share/wordlists/rockyou.txt --force

Results

<img width="1229" height="456" alt="3 2 2_hash_identification" src="https://github.com/user-attachments/assets/c73e171b-6d16-426b-b529-f66c164562d7" />

Observation

Hashcat initialized successfully and detected the available processing device before attempting to load the target credential hash.

However, the attack failed and returned the following error:

Plain Text
Token length exception
No hashes loaded

The supplied hash was rejected before the auditing process could begin.

Analysis

Investigation of the account credential revealed that Kali Linux stored passwords using the:

$y$

prefix, which corresponds to the yescrypt password-hashing algorithm.

The Hashcat attack was executed using:

-m 1800

which corresponds to:

SHA512crypt ($6$)

Because the supplied credential hash and selected Hashcat attack mode represented different hashing algorithms, Hashcat rejected the input file and terminated the attack.

This outcome demonstrated one of the most important concepts in password auditing:

Hash Acquisition
↓
Hash Identification
↓
Attack Mode Selection
↓
Password Auditing

Successful credential auditing requires not only obtaining password hashes but also correctly identifying the underlying hashing algorithm before selecting an attack configuration.

Section B Conclusion

The Hashcat assessment demonstrated the importance of hash identification and tool compatibility during credential auditing. Although the extracted password hashes were successfully obtained from the authentication database, the selected attack mode was incompatible with the modern yescrypt hashing algorithm used by Kali Linux.

The resulting failure highlighted a practical challenge frequently encountered during password audits: obtaining password hashes alone is insufficient without a proper understanding of the hashing mechanisms protecting those credentials.

The exercise also reinforced the importance of modern password-storage technologies. By utilizing the yescrypt hashing algorithm, Kali Linux introduces an additional layer of resistance against offline password-auditing attacks and increases the complexity of successful credential recovery.

Project Conclusion
Password Policy Configuration and Credential Auditing

This project successfully demonstrated the implementation, enforcement, and evaluation of modern password-security controls within a Kali Linux environment. The assessment was divided into three major phases: password-complexity enforcement, account lifecycle management, and credential auditing.

During Phase 1, Password Authentication Modules (PAM) were configured using the libpam-pwquality module to enforce password-complexity requirements. System-wide password policies were implemented to require minimum password length, character diversity, repetition restrictions, and username-similarity prevention. Validation testing confirmed that non-compliant passwords were automatically rejected before being committed to the authentication database.

During Phase 2, account lifecycle management policies were implemented through the Linux chage utility. Password-expiration intervals, minimum password age requirements, warning periods, inactivity controls, and mandatory password-change mechanisms were successfully configured and verified. Live testing demonstrated that administrative password-reset policies were enforced during authentication and that users could not continue normal account activity until password-renewal requirements were satisfied.

During Phase 3, credential auditing was performed using John the Ripper and Hashcat. Authentication databases were extracted, password hashes were prepared using the unshadow utility, and multiple password-auditing methodologies were explored. John the Ripper successfully processed the extracted hashes and demonstrated Single Crack Mode, Rule-Based Candidate Generation, and Dictionary-Based attacks. Despite extensive candidate generation, no passwords were recovered, indicating strong resistance against common password-cracking techniques.

Additional testing with Hashcat highlighted the importance of accurate hash identification and attack-mode selection. Investigation revealed that modern Kali Linux systems store passwords using the yescrypt hashing algorithm, introducing compatibility considerations during credential auditing and emphasizing the importance of understanding password-storage technologies.

Overall, the project demonstrated that effective password security is achieved through multiple defensive layers rather than a single control. Password-complexity enforcement, password-aging policies, username-similarity restrictions, modern password-hashing algorithms, and continuous credential auditing collectively contribute to a stronger authentication framework. The assessment confirmed that properly configured security controls significantly increase resistance against common password-cracking and credential-compromise attacks while providing measurable improvements to system security posture.

Key Security Takeaways
-Password complexity requirements significantly increase resistance to common password attacks.
-Username-derived passwords remain highly vulnerable to rule-based auditing techniques.
-Password aging policies reduce the risk of long-lived compromised credentials.
-Modern Linux systems utilize yescrypt hashing for stronger password protection.
-Offline credential auditing highlights weaknesses before attackers can exploit them.
-Multiple layers of security controls provide stronger protection than any individual mechanism.

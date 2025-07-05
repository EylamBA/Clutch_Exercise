## Clutch_Exercise

# Reqiments:
1. Instal the requests libary using pip installer. If you dont already have the libary installed, Run this command in the terminal: ```pip install requests```
2. Have a txt file named **GITHUB_PAT.txt** that will contain your Okta users PATs from GitHub
3. Have a txt file named **OKTA_DATA.txt** that will contain your Okta dev domain in the format: OKTA_DOMAIN dev-xxxxxxxx.okta.com
4. Have a txt file named **TOKENS.txt** that will contain your Okta API token in the format: OKTA_API_TOKEN **YOUR API TOKEN**
5. After you completed all the steps above, you can run the script named **script.py**
6. When you run the script, it will print when it finished certain steps, for example:
```
Added GitHub data of benarieylam-user1_ClutchEx
Added GitHub data of benarieylam-user2_ClutchEx
Added GitHub data of benarieylam-user3_ClutchEx
Added Okta data of benarieylam+user3_at_gmail.com
Added Okta data of benarieylam+user1_at_gmail.com
Added Okta data of benarieylam+user2_at_gmail.com
Added unified data of user1
Added unified data of user2
Added unified data of user3
```
7. In the directory **Data_Gathered** you would see Github- and Okta- file, each one is realted to it's platform.
8. In the directory **Unified_Files** you would see files name after the username in the Okta platform, and in the file you will see the unified data from GitHub and Okta.

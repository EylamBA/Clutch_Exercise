import requests
import time 
import ast
import os

#GitHub Url that works with the API
GITHUB_API_URL = "https://api.github.com/user"
GITHUB_PAT_FILE = "GITHUB_PAT.txt"

OKTA_DOMAIN = ""
OKTA_API_TOKEN = ""
GROUP_NAME = 'Clutch'

DIRECTORY_TO_SAVE_DATA_GATHERD = "Data_Gathered"
GITHUB_RELATED_FILES = "GitHub-"
OKTA_RELATED_FILES = "Okta-"

def getGitHubUserDataUsingPAT():
    """
    The function get the PATS from the PATS file and get the data of each user from it PAT
    """

    # getting the PAT from the file
    PATs = []
    pats_file = open("GITHUB_PAT.txt", "r")
    pats_lines = pats_file.readlines()
    for pat in pats_lines:
        pat = pat.rstrip('\n')
        PATs.append(pat)


    for pat in PATs:
        # cretaing the request of the data from the PAT
        headers = {
        "Authorization": f"token {pat}",
        "Accept": "application/vnd.github+json"
        }
        response = requests.get(GITHUB_API_URL, headers=headers)

        # parsing the data got from the request into json format
        user_data = response.json()

        #cretaing a file for each user that contains the information about the user
        login_data =  user_data["login"]
        output_filename = GITHUB_RELATED_FILES + login_data + ".txt"
        with open(f"{DIRECTORY_TO_SAVE_DATA_GATHERD}\{output_filename}", 'w') as f:
            f.write(f"PAT: {pat}\n")
            for key, value in user_data.items():
                f.write(f"{key}: {value}\n")
        
        print(f"Added GitHub data of {login_data}")
        time.sleep(0.2)


def getOktaDetails():
    """
    Get the OKTA domain and the okta api token from the files
    """

    # declering the global varibles
    global OKTA_DOMAIN
    global OKTA_API_TOKEN

    # getting the okta domain from the okta data file
    okta_file = open("OKTA_DATA.txt", "r")
    okta_domain = okta_file.readline()
    okta_domain = okta_domain.split(" ")[1]
    OKTA_DOMAIN = f'https://{okta_domain}'

    # get the API token of the okta user from the token file
    tokens = {}
    tokens_file = open("TOKENS.txt", "r")
    tokens_lines = tokens_file.readlines()
    for line in tokens_lines:
        line = line.rstrip('\n')
        line = line.split(" ")
        tokens[line[0]] = line[1]
    
    OKTA_API_TOKEN = tokens["OKTA_API_TOKEN"]


def getGroupIdInOkta():
    """
    Get the Gruop ID of the users that also in i the GitHub organization

    Returns:
        string: the group id of the selected group
    """
    
    # The header of the request, containd the OKTA user API token
    headers = {
        'Authorization': f'SSWS {OKTA_API_TOKEN}',
        'Accept': 'application/json'
    }
    
    # do a get request of the group id and return it
    group_search_url = f'{OKTA_DOMAIN}/api/v1/groups?q={GROUP_NAME}'
    group_response = requests.get(group_search_url, headers=headers)
    groups = group_response.json()
    group_id = groups[0]['id']
    return group_id

def getUsersFromThegroup(group_id):
    # The header of the request, containd the OKTA user API token
    headers = {
        'Authorization': f'SSWS {OKTA_API_TOKEN}',
        'Accept': 'application/json'
    }

    # the request for the users
    group_users_url = f'{OKTA_DOMAIN}/api/v1/groups/{group_id}/users'
    users_list = []
    next_url = group_users_url

    while next_url:
        # parsing the response and adding it to the users list
        response = requests.get(next_url, headers=headers)
        users_list.extend(response.json())

        # cheking if there is another page of users
        if 'next' in response.links:
            next_url = response.links['next']['url']
        else:
            next_url = None
    
    return users_list

def writingTheDataToFiles(users_list):
    headers = {
        'Authorization': f'SSWS {OKTA_API_TOKEN}',
        'Accept': 'application/json'
    }

    for user in users_list:
        # create the folder name of the user
        uid = user['id']
        login = user['profile'].get('login', uid).replace('@', '_at_')
        filename = f'{OKTA_RELATED_FILES}{login}.txt'
        lines = []

        # getting full user details
        r = requests.get(f'{OKTA_DOMAIN}/api/v1/users/{uid}', headers=headers);
        user_full = r.json()
        for key, value in user_full.items():
                lines.append(f"{key}: {value}")
        #lines.append(f"{user_full}\n")

        # getting assigned apps of teh user
        apps_url = f'{OKTA_DOMAIN}/api/v1/apps?filter=user.id+eq+"{uid}"'
        r = requests.get(apps_url, headers=headers);
        apps = r.json()
        for app in apps:
            for key, value in app.items():
                lines.append(f"{key}: {value}")
        #lines.append(f"{apps}\n")

        # Write to file
        with open(f"{DIRECTORY_TO_SAVE_DATA_GATHERD}\{filename}", 'w') as f:
            f.write("\n".join(lines))
        
        print(f"Added Okta data of {login}")

        time.sleep(0.2)

def getOktaUsersData():
    """
    Getting the data of the OKTA users
    """
    getOktaDetails()
    group_id = getGroupIdInOkta()
    users_list = getUsersFromThegroup(group_id)
    writingTheDataToFiles(users_list)


def extractFiledsFromGitHubData(github_user_data_file_path):
    """
    Get the desegnatied fiels from the github data file

    Args:
        github_user_data_file_path (string): the path to the github file

    Returns:
        dict: contains all the key and values from the github file
    """

    github_data = {}

    # keys to take from the file
    keys_to_gather = ["PAT", "login", "id", "name", "email", "notification_email", "created_at"]

    with open(github_user_data_file_path, 'r') as f:
        for line in f:
            line = line.strip()
            # skip empty lines
            if not line:
                continue

            # split key and value
            if ':' not in line:
                continue  

            # get key and value
            key, value = line.split(':', 1)
            key = key.strip()
            value = value.strip()

            # parse only PAT, login, id, name, email, notification_email, created_at
            if key in keys_to_gather and key not in github_data:
                github_data[key] = value

    return github_data

def extractFiledsFromOktaData(okta_user_data_file_path):
    """
    Get the desegnatied fiels from the okta data file

    Args:
        okta_user_data_file_path (string): the path to the okta file

    Returns:
        dict: contains all the key and values from the okta file
    """

    okta_data = {}

    # keys to take from the file
    keys_to_gather = ['id', 'status', 'created', 'activated', 'label']

    with open(okta_user_data_file_path, 'r') as f:
        for line in f:
            line = line.strip()
            # skip empty lines
            if not line:
                continue

            # split key and value
            if ':' not in line:
                continue  

            # get key and value
            key, value = line.split(':', 1)
            key = key.strip()
            value = value.strip()

            # parse only id, status, created, activated, label, profile(firstName, lastName, login, email) data
            if key in keys_to_gather and key not in okta_data:
                okta_data[key] = value
            elif key == 'profile':
                try:
                    profile_dict = ast.literal_eval(value)
                    okta_data['firstName'] = profile_dict.get('firstName', '')
                    okta_data['lastName'] = profile_dict.get('lastName', '')
                    okta_data['login'] = profile_dict.get('login', '')
                    okta_data['email'] = profile_dict.get('email', '')
                except Exception:
                    pass
    
    return okta_data


def unifiedData():
    """
    gather the data from all the files and unified the ones who realted to the same user
    """

    # go through every file and gather the data needed
    github_data = []
    okta_data = []
    for data_file in os.listdir(DIRECTORY_TO_SAVE_DATA_GATHERD):
        path = os.path.join(DIRECTORY_TO_SAVE_DATA_GATHERD, data_file)
        if GITHUB_RELATED_FILES in path:
            github_data.append(extractFiledsFromGitHubData(path))
        if OKTA_RELATED_FILES in path:
            okta_data.append(extractFiledsFromOktaData(path))
    
    # check for same user on the diffrent file
    for g_user in github_data:
        for o_user in okta_data:
            if g_user["notification_email"] == o_user["email"]:
                filename = o_user["firstName"] + ".txt"

                # open a file and write the data that related to the user from github and okta
                with open(f"Unified_Files\{filename}", 'w') as f:
                    pat = g_user["PAT"]
                    first_name = o_user["firstName"]
                    f.write(f"Okta user {first_name} have PAT: {pat}\n\n")

                    f.write("GitHub data:\n")
                    for key, value in g_user.items():
                        f.write(f"{key}: {value}\n")

                    f.write("\nOkta data:\n")
                    for key, value in o_user.items():
                        f.write(f"{key}: {value}\n")

            
                print(f"Added unified data of {first_name}")




def main():
    getGitHubUserDataUsingPAT()
    getOktaUsersData()
    unifiedData()


if __name__ == "__main__":
    main()
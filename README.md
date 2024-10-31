# Google-drive-api-download-folder

https://developers.google.com/oauthplayground/
每次获取特定权限的链接是固定的
```
https://accounts.google.com/o/oauth2/v2/auth/oauthchooseaccount?redirect_uri=https%3A%2F%2Fdevelopers.google.com%2Foauthplayground&prompt=consent&response_type=code&client_id=407408718192.apps.googleusercontent.com&scope=https%3A%2F%2Fwww.googleapis.com%2Fauth%2Fdrive%20https%3A%2F%2Fwww.googleapis.com%2Fauth%2Fdrive.file%20https%3A%2F%2Fwww.googleapis.com%2Fauth%2Fdrive.readonly%20https%3A%2F%2Fwww.googleapis.com%2Fauth%2Fdrive.scripts&access_type=offline&service=lso&o2v=2&ddm=0&flowName=GeneralOAuthFlow
```

scrops要网站管理自己手动拼凑？

链接中参数 scopes

scrops=https://www.googleapis.com/auth/drive%20https://www.googleapis.com/auth/drive.file%20https://www.googleapis.com/auth/drive.readonly

登录之后获取access_token，将其作为headers中，或 -H "Authorization: Bearer xxxxxxx 的样式，的cookie使用

命令行命令：

```shell
%%shell
#!/bin/bash

# 定义变量
ACCESS_TOKEN="ya29.RX5jqVTJ8Gk-IvKtB1OHjbP_LmPx_oUnwwGXa7d_yaJxJJPMcASbGIAwuDF73ZkUiMAA1Z61AAFLm0fWLdMCILwfruyp_CM5UiofAqQH7DfErMuomsnLnQ_t6fYNxUvrlKrVI_VoIF4ZQ5ugQqQNSDnk7vB3TaAZjeqfiMH9lgjNXnUYj9kbPT8WqqoVsPAVCBqJCfT3h_lYYAPda4xuUzxV_u" 
OUTPUT_FILE="folder_list.txt"
NEXT_PAGE_TOKEN=""
PARENT_ID=" 
5h1BrOyT-9JeBagxeABq-rElQaLjhIvSv
 " #文件夹id
# 初始化输出文件
> "$OUTPUT_FILE"

# 循环获取所有页面
RESPONSE=$(curl -s -X GET "https://www.googleapis.com/drive/v3/files?q='${PARENT_ID}'+in+parents&pageToken=${NEXT_PAGE_TOKEN}" \
  -H "Authorization: Bearer $ACCESS_TOKEN")
echo "$RESPONSE"
  # 提取文件名并保存到文件
echo "$RESPONSE" | jq -r '.files[] | select(.mimeType != "application/vnd.google-apps.folder") | .name' >> "$OUTPUT_FILE"

  # 提取nextPageToken
NEXT_PAGE_TOKEN=$(echo "$RESPONSE" | jq -r '.nextPageToken // empty')


echo "Folder list saved to $OUTPUT_FILE"
```

或者python：

```python
import requests

# 定义变量
ACCESS_TOKEN = "ya29.-"
NEXT_PAGE_TOKEN = "" 
PARENT_ID = ""  # 如果您要列出特定文件夹的内容，请在这里设置文件夹的ID

# 循环获取所有页面
while True:
    # 获取文件列表
    params = {
        'q': f"'{PARENT_ID}' in parents and mimeType != 'application/vnd.google-apps.folder'",
        'pageToken': NEXT_PAGE_TOKEN,
        'fields': 'nextPageToken, files(name)'
    }
    headers = {
        'Authorization': f'Bearer {ACCESS_TOKEN}'
    }
    response = requests.get('https://www.googleapis.com/drive/v3/files', params=params, headers=headers)

    # 解析响应
    data = response.json()
    files = data.get('files', [])

    # 打印文件名
    for file_data in files:
        print(file_data['name'])

    # 检查是否有下一页
    NEXT_PAGE_TOKEN = data.get('nextPageToken', '')
    if not NEXT_PAGE_TOKEN:
        break
```

打印内容保存到txt，
检查文件夹中在txt中的文件

```
while IFS= read -r file_name; do
    if [ -f "/storage/emulated/0/okk (1)/$file_name" ]; then
        echo "$file_name"
    fi
done < "/storage/emulated/0/okk (1)/is.txt"
```

检查文件夹内文件是否都在txt中

```
while IFS= read -r file_name; do
    if [ -f "/storage/emulated/0/Download/2f933f24eb93f9b1b68de52e678e0633/$file_name" ]; then
        if grep -q "^$file_name$" "/storage/emulated/0/okk (1)/n.txt"; then
            echo "Found: $file_name"
        else
            echo -e "\033[31mNot found in n.txt: $file_name\033[0m"
        fi
    else
        echo -e "\033[31mFile does not exist: $file_name\033[0m"
    fi
done < <(find "/storage/emulated/0/Download/2f933f24eb93f9b1b68de52e678e0633/" -type f -maxdepth 1 -exec basename {} \;)
```

```
# 首先创建目标文件夹，如果它不存在的话
mkdir -p "/storage/emulated/0/okk (2)"

# 然后使用 while 循环读取文件名，并移动文件
while IFS= read -r file_name; do
    if [ -f "/storage/emulated/0/okk (1)/$file_name" ]; then
        echo "Moving $file_name to /storage/emulated/0/okk (2)"
        mv "/storage/emulated/0/okk (1)/$file_name" "/storage/emulated/0/okk (2)/"
    else
        echo "File $file_name does not exist in /storage/emulated/0/okk (1)"
    fi
done < "/storage/emulated/0/okk (1)/is.txt"
```

## 下载谷歌网盘，的分享链接文件夹：

### 1. 获取文件列表:

输出格式:
`[('文件下载id', '文件路径'), (None, '365 Days'), ('1Be2IAwPdMO35D0dsdoU0c5X9RP1W7y1U', '365 Days/2000x3000.png'), ('1VFLIe6RNSZHvcoQ_F21dCHRNDmESBGhM', '365 Days/683x1024.png'), ]`
这其实就是以下文件树：

 ```
├── 365 Days
│
│   ├── 2000x3000.png
│   └── 683x1024.png
```
 
上方None，指的是文件夹
上方文件id，其实就是，他人分享的网页链接 https://drive.google.com/drive/folders/1XJZ3_ajeUOiZkVJfp9DutBBLEQUF2UdJ?usp=drive_link 中，后面这一串全部 `1XJZ3_ajeUOiZkVJfp9DutBBLEQUF2UdJ` 这个id，这个id文件夹有，文件夹内每个文件也有。打开子文件夹时候，其实网页链接就是这个文件夹id，点击查看单个文件时候，我这里是"在新视窗中检视" 链接也直接就是文件id。说白了id就是网盘分享链接。

文件id可以通过以下两种方式下载：

```

https://drive.google.com/uc?export=download&id=文件id
```

```
https://www.googleapis.com/drive/v3/files/文件id?alt=media
```

```python
from bs4 import BeautifulSoup
import requests
import json
import re
from itertools import islice  # 添加这行以导入islice
import os.path as osp

client = requests.session()

folders_url = "https://drive.google.com/drive/folders/"
files_url = "https://drive.google.com/uc?id="
folder_type = "application/vnd.google-apps.folder"

string_regex = re.compile(r"'((?:[^'\\]|\\.)*)'")

MAX_NUMBER_FILES = 50

class GoogleDriveFile(object):
    def __init__(self, id, name, type, children=None):
        self.id = id
        self.name = name
        self.type = type
        self.children = children if children is not None else []

    def is_folder(self):
        return self.type == folder_type

def parse_google_drive_file(folder, content):
    folder_soup = BeautifulSoup(content, features="html.parser")

    encoded_data = None
    for script in folder_soup.select("script"):
        inner_html = script.decode_contents()
        if "_DRIVE_ivd" in inner_html:
            regex_iter = string_regex.finditer(inner_html)
            try:
                encoded_data = next(islice(regex_iter, 1, None)).group(1)
            except StopIteration:
                raise RuntimeError("无法找到文件夹编码的JS字符串")
            break

    if encoded_data is None:
        raise RuntimeError("未找到包含_DRIVE_ivd的script标签")

    decoded = encoded_data.encode("utf-8").decode("unicode_escape")
    folder_arr = json.loads(decoded)

    folder_contents = [] if folder_arr[0] is None else folder_arr[0]

    gdrive_file = GoogleDriveFile(
        id=folder[39:],
        name=folder_soup.title.contents[0][:-15],
        type=folder_type,
    )

    id_name_type_iter = ((e[0], e[2], e[3]) for e in folder_contents)

    return gdrive_file, id_name_type_iter

def download_and_parse_google_drive_link(folder):
    folder_page = client.get(folder)

    if folder_page.status_code != 200:
        return False, None

    gdrive_file, id_name_type_iter = parse_google_drive_file(folder, folder_page.text)

    for child_id, child_name, child_type in id_name_type_iter:
        if child_type != folder_type:
            gdrive_file.children.append(
                GoogleDriveFile(
                    id=child_id,
                    name=child_name,
                    type=child_type,
                )
            )
            continue

        return_code, child = download_and_parse_google_drive_link(folders_url + child_id)
        if not return_code:
            return return_code, None
        gdrive_file.children.append(child)

    return True, gdrive_file

def get_directory_structure(gdrive_file, previous_path):
    directory_structure = []
    for file in gdrive_file.children:
        if file.is_folder():
            directory_structure.append(
                (None, osp.join(previous_path, file.name))
            )
            for i in get_directory_structure(file, osp.join(previous_path, file.name)):
                directory_structure.append(i)
        elif not file.children:
            directory_structure.append(
                (file.id, osp.join(previous_path, file.name))
            )
    return directory_structure

def get_folder_structure(url):
    return_code, gdrive_file = download_and_parse_google_drive_link(url)
    if not return_code:
        return None
    return get_directory_structure(gdrive_file, "")

# 替换为提供的网盘分享链接URL
url = "https://drive.google.com/drive/u/0/mobile/folders/1XJZ3_ajeUOiZkVJfp9DutBBLEQUF2UdJ?usp=drive_link"
structure = get_folder_structure(url)
print(structure)
```

运行完第一步后，继续运行第二步，这两个代码可以拼在一起运行
### 2. 下载文件

```python
import os
import requests
headers = {
    'Authorization': 'Bearer ya29.a0AcM612xM7ZMR_8Q2VdKfCIZCA9PNtz', 
} #这里是access_token ，填写在Bearer空格 后面

# 假设 structure 是从 get_folder_structure 函数返回的目录结构
def download_file(file_id, file_path):
    url = f"https://www.googleapis.com/drive/v3/files/{file_id}?alt=media"
    response = requests.get(url, headers=headers)
    if response.status_code == 200:
        os.makedirs(os.path.dirname(file_path), exist_ok=True)
        with open(file_path, 'wb') as f:
            f.write(response.content)
        print(f"Downloaded {file_path}")
    else:
        print(f"Failed to download {file_path}")

def download_files(structure):
    for file_id, file_path in structure:
        if file_id is not None:  # 文件
            download_file(file_id, file_path)

# 开始下载文件
download_files(structure)
```

参考：
1 . https://stackoverflow.com/questions/24720075/how-to-get-list-of-files-by-folder-on-google-drive-api

2 . gdown的下载文件夹部分 ： https://github.com/wkentaro/gdown/blob/main/gdown/download_folder.py

3 . 获取access_token :
https://developers.google.com/admob/api/v1/how-tos/playground

4 . python drive api 申请api :
https://developers.google.com/drive/api/quickstart/python

5 . 删除access_token :
https://stackoverflow.com/questions/55529305/how-to-disconnect-remove-access-of-google-drive-using-google-apis-drive-v3/55529980#55529980

https://developers.google.com/identity/protocols/oauth2/web-server#python_8

```python
requests.post('https://oauth2.googleapis.com/revoke',
    params={'token': credentials.token}, #credentials.token换成access_token
    headers = {'content-type': 'application/x-www-form-urlencoded'})

```

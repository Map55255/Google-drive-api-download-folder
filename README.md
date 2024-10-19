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

```
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

```import requests

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

#!/bin/bash

CURL_OPTS="-sS"
#CURL_OPTS="-v"
if [[ $CURL_PROXY ]]; then
  CURL_OPTS="$CURL_OPTS --insecure --proxy $CURL_PROXY"
fi
BOX_APP=https://app.box.com
API_PATH=https://api.box.com/2.0
AUTH_PATH=https://www.box.com/api/oauth2
AUTH_PATH_APP=$BOX_APP/api/oauth2
CLIENT_ID=TODO_YOUR_CLIENT_ID
CLIENT_SECRET=TODO_YOUR_CLIENT_SECRET
USER_ID=TODO_YOUR_URL_ENCODED_LOGIN
USER_SECRET=TODO_YOUR_URL_ENCODED_PASSWORD
NORMAL=$(tput sgr0)
GREEN=$(tput setaf 2; tput bold)
YELLOW=$(tput setaf 3)

[[ $(type -p brew) ]] || ruby -e "$(curl -fsSL https://raw.github.com/mxcl/homebrew/go)"
[[ $(type -p jq) ]] || brew install jq
[[ $(type -p nc) ]] || ( echo "nc must be installed"; exit 1 )

function green() { 
  echo -e "$GREEN$*$NORMAL" 
}

function yellow() {
  echo -e "$YELLOW$*$NORMAL"
}

# mostly dump responses to files to ease debugging

function _get_request_token {
  curl -L $CURL_OPTS "$AUTH_PATH/authorize?response_type=code&client_id=${CLIENT_ID}&state=authenticated" \
     > /tmp/get_request_token.html
  egrep -o "\"request_token\" value=\"[a-z0-9]*\"" < /tmp/get_request_token.html | cut -d= -f2 | cut -d'"' -f2
}

function _get_ic_code {

local_request_token=$1
curl --include -L $CURL_OPTS \
  -X POST \
  "$AUTH_PATH_APP/authorize?response_type=code&client_id=${CLIENT_ID}&state=authenticated" \
-d \
"login=${USER_ID}"\
"&password=${USER_PASSWORD}"\
"&login_submit=Authorizing..."\
"&dologin=1"\
"&client_id=${CLIENT_ID}"\
"&response_type=code&"\
"redirect_uri=http%3A%2F%2Flocalhost%3A17123"\
"&scope=root_readwrite"\
"&folder_id="\
"&state=authenticated"\
"&reg_step="\
"&submit1=1"\
"&folder="\
"&login_or_register_mode=login"\
"&new_login_or_register_mode="\
"&__login=1"\
"&_redirect_url=%2Fapi%2Foauth2%2Fauthorize%3Fresponse_type%3Dcode%26client_id%3D${CLIENT_ID}%26state%3Dauthenticated"\
"&request_token=${local_request_token}&_pw_sql=" \
> /tmp/get_ic_code.html

egrep -o "\"ic\" value=\"[a-z0-9]*\"" < /tmp/get_ic_code.html | \
cut -d= -f2 | cut -d'"' -f2
}

# bit dodgy, we depend on get_ic_code existing
function _get_magic_cookie {
  egrep "Set-Cookie" < /tmp/get_ic_code.html | egrep -o "z=[A-z0-9]*" | cut -d= -f2
}

# extrac the authentication code from the redirect
function _get_auth_code {
  local_request_token=$1
  local_ic_code=$(_get_ic_code $local_request_token)
  local_magic_cookie=$(_get_magic_cookie)

curl --include $CURL_OPTS \
  -X POST \
  "$AUTH_PATH_APP/authorize?response_type=code&client_id=${CLIENT_ID}&state=authenticated" \
  --cookie "z=${local_magic_cookie}" \
  -d \
"client_id=${CLIENT_ID}"\
"&response_type=code"\
"&redirect_uri=http%3A%2F%2Flocalhost%3A17123"\
"&scope=root_readwrite"\
"&folder_id="\
"&state=authenticated"\
"&doconsent=doconsent"\
"&ic=${local_ic_code}"\
"&consent_accept=Grant+access+to+Box" > /tmp/get_auth_code.html

 egrep Location < /tmp/get_auth_code.html | egrep -o "code=[A-z0-9]*" | cut -d= -f2
}



# Authorise with Box in one easy function.
function box_authorize {

request_token=$(_get_request_token)
auth_code=$(_get_auth_code $request_token)

curl -L $CURL_OPTS "$AUTH_PATH/token" \
  -d "grant_type=authorization_code&code=${auth_code}&client_id=${CLIENT_ID}&client_secret=${CLIENT_SECRET}" \
  -X POST \
  | jq '.access_token, .refresh_token' | tr -d '"' | tr "\n" " "
}

function box_revoke {
curl $CURL_OPTS $AUTH_PATH/revoke \
  -d "client_id=${CLIENT_ID}&client_secret=${CLIENT_SECRET}&token=${ACCESS_TOKEN}" \
  -X POST
echo "Revoked $ACCESS_TOKEN"
unset ACCESS_TOKEN
}

if [[ -z $USER_TOKEN && $ACCESS_TOKEN ]]; then
  echo "Revoking $ACCESS_TOKEN"
  box_revoke
fi

# export USER_TOKEN in your environment to use an existing 
# token, eg, USER_TOKEN=MY_TOKEN . box_api
IFS=' ' read _access_token REFRESH_TOKEN <<< $(box_authorize)

ACCESS_TOKEN=${USER_TOKEN:-$_access_token}

function box_folder_items {
  folder=${1:-0}
  curl $CURL_OPTS $API_PATH/folders/$folder/items \
    -H "Authorization: Bearer $ACCESS_TOKEN" | jq '.' 
}

function box_thumbnail_request {
  file_id=$1
  min_width=${2-128}
  min_height=${3-$min_width}
  tmp_file=$(mktemp -t box_api)
  curl -isSL "$API_PATH/files/$file_id/thumbnail.png?min_height=${min_height}&min_width=${min_width}" \
    -H "Authorization: Bearer $ACCESS_TOKEN"

}

function box_thumbnail {
  file_id=$1
  min_width=${2-256}
  min_height=${3-$min_width}
  tmp_file=$(mktemp -t box_api)
  curl -isSL "$API_PATH/files/$file_id/thumbnail.png?min_height=${min_height}&min_width=${min_width}" \
    -H "Authorization: Bearer $ACCESS_TOKEN" > $tmp_file
  status=$(head -1 $tmp_file | cut -d' ' -f2)

  if [[ $status = "202" ]]; then
    echo "Waiting for thumb generation"
    sleep 3  
  fi
  echo "Writing to thumb_$file_id.png"
  curl $CURL_OPTS "$API_PATH/files/$file_id/thumbnail.png?min_height=${min_height}&min_width=${min_width}" \
    -H "Authorization: Bearer $ACCESS_TOKEN" -o "thumb_$file_id.png"
}

function box_file {
  file_id=$1
  curl $CURL_OPTS "$API_PATH/files/$file_id/content" \
    -H "Authorization: Bearer $ACCESS_TOKEN"
}

green "box_api is ready!"
echo 
echo "  USER_TOKEN=$ACCESS_TOKEN"
echo "  REFRESH_TOKEN=$REFRESH_TOKEN"
echo
green "The following functions are available:"
for f in $(egrep '\<function box_' box_api | cut -d' ' -f2 | sed -e '$ d'); do
  yellow "  $f"
done


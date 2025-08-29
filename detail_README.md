# Creating a Dockerfile and testing locally
1. Create a new repo and create below Dockerfile
2. Add below content to Dockerfile
```
FROM alpine

RUN apk add --no-cache \
        bash           \
        httpie         \
        jq &&          \
        which bash &&  \
        which http &&  \
        which jq

COPY entrypoint.sh /usr/local/bin/entrypoint.sh
COPY sample_push_event.json /sample_push_event.json

ENTRYPOINT ["entrypoint.sh"]
```
3. Clone the repo and run below commands
```
docker build --tag keyword-release .
```
Running Docker build with --tag switch tells Docker to tag the image with the name keyword-release.
The dot at the end of the command tells Docker to build the image using the files inside the directory wher the command is being run.

# Running the script
1. Make the script executable locally by running below command:
   ```
   chmod +x entrypoint.sh
   ```
2. entrypoint.sh file:
```
#!/bin/bash
set -e

if [ -n "$GITHUB_EVENT_PATH" ];
then
    EVENT_PATH=$GITHUB_EVENT_PATH
elif [ -f ./sample_push_event.json ];
then
    EVENT_PATH='./sample_push_event.json'
    LOCAL_TEST=true
else
    echo "No JSON data to process! :("
    exit 1
fi

env
jq . < $EVENT_PATH

# if keyword is found
if jq '.commits[].message, .head_commit.message' < $EVENT_PATH | grep -i -q "$*";
then
    # do something
    VERSION=$(date +%F.%s)

    DATA="$(printf '{"tag_name":"v%s",' $VERSION)"
    DATA="${DATA} $(printf '"target_commitish":"master",')"
    DATA="${DATA} $(printf '"name":"v%s",' $VERSION)"
    DATA="${DATA} $(printf '"body":"Automated release based on keyword: %s",' "$*")"
    DATA="${DATA} $(printf '"draft":false, "prerelease":false}')"

    URL="https://api.github.com/repos/${GITHUB_REPOSITORY}/releases?access_token=${GITHUB_TOKEN}"

    if [[ "${LOCAL_TEST}" == *"true"* ]];
    then
        echo "## [TESTING] Keyword was found but no release was created."
    else
        echo $DATA | http POST $URL | jq .
    fi
# otherwise
else
    # exit gracefully
    echo "Nothing to process."
fi
```

- $ is a variable that contains the arguments that were passed to the script. We are echoing those arguments to a grep command that will look for the word fixed.
- The -i switch tells grep to ignore a case.
- The -q switch tells grep to be quiet, so no output gets printed.
- VERSION : used to name and tag the release
- On the next few lines the VERSION variable is used along with other printf statements to format a JSON string and store that in a variable DATA.
- The DATA will be posted to the Github API to create the release.
- In the next line, the API URL is formed by using the GITHUB_REPOSITORY and GITHUB_TOKEN environment variable to authenticate to our Github Account.
- The LOCAL_TESTvariable is used to add some logic.
    - If set to TRUE, then we won’t post to the Github API since the environment won’t have a token and a Github Repository won’t be set
    - If set FALSE, then data would be posted to the API
- To post the data, it is piped to the HTTP command that will post it to the URL created above. This command would return a JSON output hence piping that to JQ to make it easier to read in the log.

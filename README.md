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
   ```bash
#!/bin/bash  
set -e #exit the script immediately if an error is encountered

# if keyword is found
if echo "$*" | grep -i -q FIXED; 
then
    # do something
    echo "Found keyword."
# otherwise
else
    # exit gracefully
    echo "Nothing to process."
fi
```

- $ is a variable that contains the arguments that were passed to the script. We are echoing those arguments to a grep command that will look for the word fixed.
- The -i switch tells grep to ignore a case.
- The -q switch tells grep to be quiet, so no output gets printed.

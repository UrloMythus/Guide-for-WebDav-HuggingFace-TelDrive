# Guide-for-WebDav-HuggingFace-TelDrive
##All Credits to ./rin to have helped me to set up everything. Warning: I am not gonna explain how HF works here, if you do not know how watch the previous tutorial. 
1 Go to https://huggingface.co/new-space  and create a new public Docker space 
2 Create a file called .gitattributes (Might not be needed but tbh better having it)  and put this inside: 
```
*.7z filter=lfs diff=lfs merge=lfs -text
*.arrow filter=lfs diff=lfs merge=lfs -text
*.bin filter=lfs diff=lfs merge=lfs -text
*.bz2 filter=lfs diff=lfs merge=lfs -text
*.ckpt filter=lfs diff=lfs merge=lfs -text
*.ftz filter=lfs diff=lfs merge=lfs -text
*.gz filter=lfs diff=lfs merge=lfs -text
*.h5 filter=lfs diff=lfs merge=lfs -text
*.joblib filter=lfs diff=lfs merge=lfs -text
*.lfs.* filter=lfs diff=lfs merge=lfs -text
*.mlmodel filter=lfs diff=lfs merge=lfs -text
*.model filter=lfs diff=lfs merge=lfs -text
*.msgpack filter=lfs diff=lfs merge=lfs -text
*.npy filter=lfs diff=lfs merge=lfs -text
*.npz filter=lfs diff=lfs merge=lfs -text
*.onnx filter=lfs diff=lfs merge=lfs -text
*.ot filter=lfs diff=lfs merge=lfs -text
*.parquet filter=lfs diff=lfs merge=lfs -text
*.pb filter=lfs diff=lfs merge=lfs -text
*.pickle filter=lfs diff=lfs merge=lfs -text
*.pkl filter=lfs diff=lfs merge=lfs -text
*.pt filter=lfs diff=lfs merge=lfs -text
*.pth filter=lfs diff=lfs merge=lfs -text
*.rar filter=lfs diff=lfs merge=lfs -text
*.safetensors filter=lfs diff=lfs merge=lfs -text
saved_model/**/* filter=lfs diff=lfs merge=lfs -text
*.tar.* filter=lfs diff=lfs merge=lfs -text
*.tar filter=lfs diff=lfs merge=lfs -text
*.tflite filter=lfs diff=lfs merge=lfs -text
*.tgz filter=lfs diff=lfs merge=lfs -text
*.wasm filter=lfs diff=lfs merge=lfs -text
*.xz filter=lfs diff=lfs merge=lfs -text
*.zip filter=lfs diff=lfs merge=lfs -text
*.zst filter=lfs diff=lfs merge=lfs -text
*tfevents* filter=lfs diff=lfs merge=lfs -text
```
3 Create a file called docker-entrypoint.sh and put this: 
```
#! /bin/sh -eux

mkdir -p $HOME/.config/rclone

mkdir -p $HOME/rclone/cache

if [ "$RCLONE_CONFIG_BASE64" != "" ]; then
  echo "[INFO] Config Rclone from RCLONE_CONFIG_BASE64 env"
  echo $RCLONE_CONFIG_BASE64 | base64 -d > $HOME/.config/rclone/rclone.conf
  echo "[INFO] Config Rclone from RCLONE_CONFIG_BASE64 completed"
fi

rclone serve webdav --vfs-cache-mode writes --addr :7860 --cache-dir $HOME/rclone/cache --user "$USER" --pass "$PASS" td_hf:
```

4 Create a file called Dockerfile and put this: 
```
# Download git repo
FROM ubuntu AS git-download
RUN apt-get update && apt-get install -y git
WORKDIR /app
RUN git clone https://github.com/divyam234/rclone.git .


# Build rclone
FROM golang AS rclone-builder

COPY --from=git-download /app /go/src/github.com/rclone/rclone/
WORKDIR /go/src/github.com/rclone/rclone/

RUN \
  CGO_ENABLED=0 \
  make
RUN ./rclone version


# Running Container
FROM ubuntu

WORKDIR /app

RUN apt update && apt install curl  unzip -y 

## Get rclone from builder
COPY --from=rclone-builder /go/src/github.com/rclone/rclone/rclone /usr/local/bin/

RUN useradd -m -u 1000 user

USER user

ENV HOME=/home/user \
    PATH=/home/user/.local/bin:$PATH

RUN mkdir -p $HOME/.local/bin

WORKDIR $HOME/app

COPY --chown=user . $HOME/app

RUN rclone version

EXPOSE 7860

RUN chmod 755 ./docker-entrypoint.sh

ENTRYPOINT ["./docker-entrypoint.sh"]
```
5 Now you will have to create your BASE 64 to do that go to this site: ttps://www.base64encode.org/  and   create your BASE 64 with your informations 
```
[td_hf]
type = teldrive
access_token = your cookie
api_host = your hf link
```
6 If you do not know how to get the cookie follow this: For Firefox click F12 or CNTRL+SHIFT+I and go into the storage section. From there you will see a cookie called user_session: copy it. For Chromium based browser the process is similar tho instead of going into the Storage section, go into the Application section and then into the Cookies section. 
7 Create a Secret var. In the first field insert: RCLONE_CONFIG_BASE64 In the second Field: the value that we got from before (your base64). 
8 Create another Secret var. First field: "USER" Second Field: Whatever you want as username
9 Create another Secret var. First Field: "PASS" Second Field: Whatever you want as pass

P.S. Remember you can mount the WebDav on your file system. To do that just follow the guides online. 

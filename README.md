# breeze  
![breeze](https://img.shields.io/badge/docker-autobuild-blueviolet)

A deep learning container for langchao cluster,include ssh server,jupyter-lab,code-server and proxychains

## Requirements
- Docker version >= 19.03
- Nvidia driver >= 418

## image pull
All available iamges can be found on [yaoing/breeze](https://hub.docker.com/r/yaoing/breeze/tags)  

for cv task:
`docker pull yaoing/breeze:cv`

for nlp task:
`docker pull yaoing/breeze:nlp`



## Run container

### example: nlp image
```docker run --rm -it --gpus all -p 2224:2224 -p 8884:8884 -p 9994:9994 --privileged  -v ~/apps:/home/yao/apps -v ~/jupyter:/home/yao/jupyter -v ~/dataset:/home/yao/dataset -v ~/frpc:/home/yao/frpc breeze:nlp```

- `-p`:bind local port
- `-v`:mount local

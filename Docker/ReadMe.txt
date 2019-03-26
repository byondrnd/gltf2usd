sudo docker build . -t lambda-node-build -f ./Dockerfile
sudo docker run -it --rm -v ~/docker-shared:/var/task/docker-shared lambda-node-build bash
FROM lambci/lambda-base:build

ENV PATH=/var/lang/bin:$PATH \
  LD_LIBRARY_PATH=/var/lang/lib:$LD_LIBRARY_PATH \
  AWS_EXECUTION_ENV=AWS_Lambda_nodejs8.10 \
  NODE_PATH=/opt/nodejs/node8/node_modules:/opt/nodejs/node_modules:/var/runtime/node_modules:/var/runtime:/var/task:/var/runtime/node_modules \
  npm_config_unsafe-perm=true

RUN rm -rf /var/runtime /var/lang && \
  curl https://lambci.s3.amazonaws.com/fs/nodejs8.10.tgz | tar -zx -C /

# Add these as a separate layer as they get updated frequently
RUN curl --silent --show-error --retry 5 https://bootstrap.pypa.io/get-pip.py | python && \
  pip install -U awscli boto3 aws-sam-cli==0.13.0 aws-lambda-builders==0.2.1 --no-cache-dir

CMD ["npm", "rebuild"]

RUN pip install --index-url=http://download.qt.io/snapshots/ci/pyside/5.9/latest/ pyside2 --trusted-host download.qt.io && \
  pip install PyOpenGL && \
  git clone https://github.com/PixarAnimationStudios/USD

RUN yum install -y wget && \
  wget https://github.com/Kitware/CMake/releases/download/v3.13.3/cmake-3.13.3.tar.gz && \
  tar xzf cmake-3.13.3.tar.gz && \ 
  cd cmake-3.13.3 && \
  ./bootstrap && \
  make && \
  make install && \
  cd ../ && \
  yum install -y python-devel && \
  yum install -y libXrandr && \
  yum install -y libXrandr-devel

RUN yum install -y mesa-libGL-devel && \
  yum install -y mesa-libGLU-devel && \
  yum install -y libXinerama && \
  yum install -y libXinerama-devel && \
  yum install -y libXcursor && \
  yum install -y libXcursor-devel && \
  wget https://github.com/glfw/glfw/releases/download/3.2.1/glfw-3.2.1.zip && \
  unzip glfw-3.2.1.zip && \ 
  cd glfw-3.2.1 && \
  cmake . -G "Unix Makefiles" && \
  make && \
  make install && \
  cd ../

RUN wget https://sourceforge.net/projects/glew/files/glew/2.1.0/glew-2.1.0.zip/download && \
  unzip download && \ 
  cd glew-2.1.0 && \
  make && \
  make install && \
  cd ../

RUN yum install -y libXi && \
  yum install -y libXi-devel && \
  python USD/build_scripts/build_usd.py /var/task/USD

# ENV PYTHONPATH ${PYTHONPATH}:/usr/local/USD/lib/python/ 
# ENV PATH ${PATH}:/usr/local/USD/bin

RUN pip install pyinstaller

RUN cd /var/task && \
  git clone https://github.com/byondrnd/gltf2usd.git && \
  cd gltf2usd/Source && \
  pip install numpy --target=. && \
  pip install enum34 --target=. && \
  pip install pillow --target=.


RUN cd /var/task/gltf2usd/Source && \
  pyinstaller --onefile gltf2usd.py -p /var/task/gltf2usd/Source:/usr/task/USD/lib/python/:/usr/local/USD/lib/python/

RUN /var/task/gltf2usd/Source/dist/gltf2usd -g /var/task/gltf2usd/Docker/test/input.gltf -o /var/task/gltf2usd/Docker/test/output.usdc && \
  /usr/task/USD/bin/usdzip /var/task/gltf2usd/Docker/test/output.usdz --arkitAsset /var/task/gltf2usd/Docker/test/output.usdc

# RUN python gltf2usd/Source/gltf2usd.py -g /root/shared-folder/Bed_Option01.gltf  -o /root/shared-folder/output.usdc && \
#   usdzip /root/shared-folder/ex.usdz --arkitAsset /root/shared-folder/output.usdc
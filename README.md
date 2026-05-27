# Отчёт к лабораторной работе №8
Подготовка окружения
```bash
export GITHUB_USERNAME=Toray-lab
cd ${GITHUB_USERNAME}/workspace
pushd .
source scripts/activate
```

Клонируем предыдущую работу (lab07) как основу для lab09
```bash
$ git clone https://github.com/${GITHUB_USERNAME}/lab07 lab09
Cloning into 'lab09'...
remote: Enumerating objects: 312, done.
remote: Counting objects: 100% (312/312), done.
remote: Compressing objects: 100% (147/147), done.
remote: Total 312 (delta 129), reused 308 (delta 128), pack-reused 0 (from 0)
Receiving objects: 100% (312/312), 1.88 MiB | 1.80 MiB/s, done.
Resolving deltas: 100% (129/129), done.
$ cd lab09
$ git submodule update --init
fatal: No url found for submodule path 'projects/lab07' in .gitmodules
```

Меняем удалённый репозиторий на новый lab09
```bash
$ git remote remove origin
$ git remote add origin https://github.com/${GITHUB_USERNAME}/lab09
```

Создаём Dockerfile поэтапно
```bash
$ cat > Dockerfile <<EOF
FROM ubuntu:18.04
EOF
$ cat >> Dockerfile <<EOF

RUN apt update
RUN apt install -yy gcc g++ cmake
EOF
$ cat >> Dockerfile <<EOF

COPY . print/
WORKDIR print
EOF
$ cat >> Dockerfile <<EOF

RUN cmake -H. -B_build -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=_install
RUN cmake --build _build
RUN cmake --build _build --target install
EOF
$ cat >> Dockerfile <<EOF

ENV LOG_PATH /home/logs/log.txt
EOF
$ cat >> Dockerfile <<EOF

VOLUME /home/logs
EOF
$ cat >> Dockerfile <<EOF

WORKDIR _install/bin
EOF
$ cat >> Dockerfile <<EOF

ENTRYPOINT ./demo
EOF
```

Сборка Docker-образа
```bash
$ docker build -t logger .
[+] Building 37.7s (14/14) FINISHED                      docker:default
 => [internal] load build definition from Dockerfile               0.0s
 => => transferring dockerfile: 375B                               0.0s
 => [internal] load metadata for docker.io/library/ubuntu:18.04    0.9s
 => [internal] load .dockerignore                                  0.0s
 => => transferring context: 83B                                   0.0s
 => [1/9] FROM docker.io/library/ubuntu:18.04@sha256:152dc042452c  0.1s
 => => resolve docker.io/library/ubuntu:18.04@sha256:152dc042452c  0.1s
 => [internal] load build context                                  0.0s
 => => transferring context: 11.84kB                               0.0s
 => CACHED [2/9] RUN apt update                                    0.0s
 => CACHED [3/9] RUN apt install -yy gcc g++ cmake                 0.0s
 => [4/9] COPY . print/                                            0.2s
 => [5/9] WORKDIR print                                            0.1s
 => [6/9] RUN cmake -H. -B_build -DCMAKE_BUILD_TYPE=Release -DCMA  3.0s
 => [7/9] RUN cmake --build _build                                 1.5s
 => [8/9] RUN cmake --build _build --target install                1.0s
 => [9/9] WORKDIR _install/bin                                     0.3s
 => exporting to image                                            30.0s
 => => exporting layers                                           21.1s
 => => exporting manifest sha256:eedd5b1d4ce2ea00b0bd44876f772d54  0.1s
 => => exporting config sha256:8bffc8f3a7c3fb64c23f8b0c38972c75ca  0.1s
 => => exporting attestation manifest sha256:94cc1d1f74759d7c12d7  0.1s
 => => exporting manifest list sha256:0f41e93f7d964bce5a1ccd75448  0.1s
 => => naming to docker.io/library/logger:latest                   0.0s
 => => unpacking to docker.io/library/logger:latest                8.4s

 3 warnings found (use docker --debug to expand):
 - WorkdirRelativePath: Relative workdir "print" can have unexpected results if the base image changes (line 6)
 - LegacyKeyValueFormat: "ENV key=value" should be used instead of legacy "ENV key value" format (line 10)
 - JSONArgsRecommended: JSON arguments recommended for ENTRYPOINT to prevent unintended behavior related to OS signals (line 15)
```

Просмотр образов
```bash
$ docker images
                                                    i Info →   U  In Use
IMAGE           ID             DISK USAGE   CONTENT SIZE   EXTRA
logger:latest   0f41e93f7d96        509MB          145MB    U
```

Создание локальной папки для логов
```bash
$ mkdir logs
```

Запуск контейнера с монтированием тома
```bash
$ docker run -it -v "$(pwd)/logs/:/home/logs/" logger
text1
Logged: text1
text2
Logged: text2
text3
Logged: text3
Exiting. Log saved to /home/logs/log.txt
```
Просмотр информации об образе
```bash
$ docker inspect logger
[
    {
        "Id": "sha256:0f41e93f7d964bce5a1ccd7544822041605a0ba7cc689415b77efbb6d84eff9c",
        "RepoTags": [
            "logger:latest"
        ],
        "RepoDigests": [
            "logger@sha256:0f41e93f7d964bce5a1ccd7544822041605a0ba7cc689415b77efbb6d84eff9c"
        ],
        "Comment": "buildkit.dockerfile.v0",
        "Created": "2026-05-27T18:54:29.87703975+03:00",
        "Config": {
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                "LOG_PATH=/home/logs/log.txt"
            ],
            "Entrypoint": [
                "/bin/sh",
                "-c",
                "./demo"
            ],
            "Volumes": {
                "/home/logs": {}
            },
            "WorkingDir": "/print/_install/bin",
            "Labels": {
                "org.opencontainers.image.ref.name": "ubuntu",
                "org.opencontainers.image.version": "18.04"
            }
        },
        "Architecture": "amd64",
        "Os": "linux",
        "Size": 144792626,
        "RootFS": {
            "Type": "layers",
            "Layers": [
                "sha256:548a79621a426b4eb077c926eabac5a8620c454fb230640253e1b44dc7dd7562",
                "sha256:761af8325592d3aeda011fb303009eaba0d40403dafde6c0740fc7ca512628cb",
                "sha256:eac1cb50f72c18d174ffeda85758ee66f2ecdfa0e09050e7535e6a83f9a49afa",
                "sha256:58fbbb7383a225fa5fda07d072c8a2e6d290464ebb2092812a55acf0e263d231",
                "sha256:5f70bf18a086007016e948b04aed3b82103a36bea41755b6cddfaf10ace3c6ef",
                "sha256:4e5337b98c80fd0266060b36689d72de9bf177ffebf47a3111d363705885b692",
                "sha256:9d7e93957f41b61acec1e588d01c88a970c45de58aecbe4c7485a9a200bd60b3",
                "sha256:f3d55476ca56e9941456474d64cea0c29d74c7f089567abe1bda84b0848af357",
                "sha256:5f70bf18a086007016e948b04aed3b82103a36bea41755b6cddfaf10ace3c6ef"
            ]
        },
        "Metadata": {
            "LastTagTime": "2026-05-27T15:54:51.319355644Z"
        },
        "Descriptor": {
            "mediaType": "application/vnd.oci.image.index.v1+json",
            "digest": "sha256:0f41e93f7d964bce5a1ccd7544822041605a0ba7cc689415b77efbb6d84eff9c",
            "size": 856
        },
        "Identity": {
            "Build": [
                {
                    "Ref": "07g80drskas1127tloy4t4myr",
                    "CreatedAt": "2026-05-27T18:54:59.723692819+03:00"
                }
            ]
        }
    }
]
```

Проверка логов
```bash
$ cat logs/log.txt
text1
text2
text3
```

Обновление README
```bash
sed -i 's/lab07/lab09/g' README.md
```
Настройка Travis CI
```bash
$ nano .travis.yml
language: cpp
services:
  - docker
script:
  - docker build -t logger .
```
Фиксация изменений и отправка на GitHub
```bash
$ git add Dockerfile .travis.yml
$ git commit -m "adding Dockerfile"
[main 7039bb4] adding Dockerfile
 2 files changed, 20 insertions(+)
 create mode 100644 Dockerfile
$ git push origin main 
```
Активация Travis CI
```bash
$ travis login --auto
/home/vlad/.local/share/gem/ruby/3.3.0/gems/travis-1.14.0/lib/travis/cli/command.rb:334:in `format': wrong number of arguments (given 5, expected 1..3) (ArgumentError)
        from /home/vlad/.local/share/gem/ruby/3.3.0/gems/travis-1.14.0/lib/travis/cli/command.rb:315:in `store_error'
        from /home/vlad/.local/share/gem/ruby/3.3.0/gems/travis-1.14.0/lib/travis/cli/command.rb:235:in `rescue in execute'
        from /home/vlad/.local/share/gem/ruby/3.3.0/gems/travis-1.14.0/lib/travis/cli/command.rb:200:in `execute'
        from /home/vlad/.local/share/gem/ruby/3.3.0/gems/travis-1.14.0/lib/travis/cli.rb:66:in `run'
        from /home/vlad/.local/share/gem/ruby/3.3.0/gems/travis-1.14.0/bin/travis:21:in `<top (required)>'
        from /usr/local/bin/travis:25:in `load'
        from /usr/local/bin/travis:25:in `<main>'
/home/vlad/.local/share/gem/ruby/3.3.0/gems/travis-1.14.0/lib/travis/tools/github.rb:91:in `hub_tokens': undefined method `each' for nil (NoMethodError)

        hub.fetch(host, []).each do |entry|
                           ^^^^^
        from /home/vlad/.local/share/gem/ruby/3.3.0/gems/travis-1.14.0/lib/travis/tools/github.rb:58:in `possible_tokens'
        from /home/vlad/.local/share/gem/ruby/3.3.0/gems/travis-1.14.0/lib/travis/tools/github.rb:42:in `each_token'
        from /home/vlad/.local/share/gem/ruby/3.3.0/gems/travis-1.14.0/lib/travis/tools/github.rb:37:in `with_token'
        from /home/vlad/.local/share/gem/ruby/3.3.0/gems/travis-1.14.0/lib/travis/cli/login.rb:30:in `login'
        from /home/vlad/.local/share/gem/ruby/3.3.0/gems/travis-1.14.0/lib/travis/cli/login.rb:52:in `run'
        from /home/vlad/.local/share/gem/ruby/3.3.0/gems/travis-1.14.0/lib/travis/cli/command.rb:208:in `execute'
        from /home/vlad/.local/share/gem/ruby/3.3.0/gems/travis-1.14.0/lib/travis/cli.rb:66:in `run'
        from /home/vlad/.local/share/gem/ruby/3.3.0/gems/travis-1.14.0/bin/travis:21:in `<top (required)>'
        from /usr/local/bin/travis:25:in `load'
        from /usr/local/bin/travis:25:in `<main>'
$ travis enable
Detected repository as Toray-lab/lab09, is this correct? |yes| yes
Toray-lab/lab09: enabled :)
```

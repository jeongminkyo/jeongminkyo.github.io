---
comments: true
tags: rails foremessage aws
layout: post
title: foremessage Rails server 세팅
---

##### foremessage Rails server Setting

----------

지난번에 VPC를 구축하였고, 이번에는 rails server를 setting 할려고한다. 가난한 개발자인 관계로 aws free tier인 t2.micro로 서버를 setting할려고한다. setting 항목은 다음과 같다.



server : ubuntu 16.04

ruby environment : rbenv

ruby : 2.5.0

rails : 5.2.3

webserver : nginx

WAS : passenger



##### rbenv 설치

ruby version 관리는 rbenv를 이용하여 관리한다.

```shell
# package manager apt-get update
sudo apt-get update

# dependencies required for rbenv and Ruby 
sudo apt-get install autoconf bison build-essential libssl-dev libyaml-dev libreadline6-dev zlib1g-dev libncurses5-dev libffi-dev libgdbm3 libgdbm-dev libmysqlclient-dev nodejs

# rbenv install
git clone https://github.com/rbenv/rbenv.git ~/.rbenv

# rbenv ruby-build install
git clone https://github.com/rbenv/ruby-build.git ~/.rbenv/plugins/ruby-build

echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bashrc
echo 'eval "$(rbenv init -)"' >> ~/.bashrc

source ~/.bashrc
```

Apt-get update 이 후, rbenv관련 의존성들을 install한다. `$PATH`에 rbenv를 추가한다.



##### ruby 설치

```shell
rbenv install 2.5.0
rbenv global 2.5.0

# ruby version 
ruby -v
```

ruby 2.5.0을 설치하고, global로 적용한다.



##### bundler 설치

```shell
# bundler install
gem install bundler

# ruby 2.2.4를 사용할 경우, bundler version을 명시해줘야된다.
gem install bundler -v 1.17.3
```



##### Nginx + Passenger 설치 

Nginx passenger는 공식사이트 가이드대로 설치한다.

```shell
# install PGP key and add HTTPS support for APT
sudo apt-get install -y dirmngr gnupg
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 561F9B9CAC40B2F7
sudo apt-get install -y apt-transport-https ca-certificates

# add our APT repository
sudo sh -c 'echo deb https://oss-binaries.phusionpassenger.com/apt/passenger xenial main > /etc/apt/sources.list.d/passenger.list'
sudo apt-get update

# Install Nginx + passenger
sudo apt-get install -y nginx-extras passenger


```

 `/etc/nginx/nginx.conf` 에서  `include /etc/nginx/passenger.conf;`의 주석을 해제한다.

```
# include /etc/nginx/passenger.conf;
```

 '#' 제거

```
include /etc/nginx/passenger.conf;
```



[출처]

https://www.digitalocean.com/community/tutorials/how-to-install-ruby-on-rails-with-rbenv-on-ubuntu-16-04

https://www.phusionpassenger.com/library/install/nginx/install/oss/xenial/

https://bluese05.tistory.com/47
FROM ruby:2.5.1

RUN apt-get update -qq && apt-get install -y build-essential libpq-dev nodejs

RUN mkdir /shopapp
WORKDIR /shopapp

ADD src/Gemfile /shopapp/Gemfile
ADD src/Gemfile.lock /shopapp/Gemfile.lock

RUN bundle install

ADD src /shopapp

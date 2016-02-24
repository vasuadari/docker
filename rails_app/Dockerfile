FROM ruby

RUN apt-get update -qq && apt-get install -y build-essential

# Nokogiri dependencies
RUN apt-get install -y libxml2-dev libxslt1-dev

# JS runtime dependencies
RUN apt-get install -y nodejs

RUN apt-get install -y mysql-client

RUN gem install bundler

ENV BUNDLE_PATH /bundle

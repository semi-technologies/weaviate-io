# Weaviate website

This repo contains the [Weaviate](https://weaviate.io) website, it's built using [Jekyll](https://jekyllrb.com/).

## Installing & Running

### Dependencies

- Software Dependencies
  - [Ruby](https://www.ruby-lang.org/en/documentation/installation/)

- Installing Dependencies in **Ubuntu/Debian**
  ```bash
  # update
  sudo apt update
  
  # install dependencies for installing ruby
  sudo apt install git curl libssl-dev libreadline-dev zlib1g-dev autoconf bison build-essential libyaml-dev libreadline-dev libncurses5-dev libffi-dev libgdbm-dev
  
  # install rbenv[ruby version manager] from its official repo
  curl -fsSL https://github.com/rbenv/rbenv-installer/raw/HEAD/bin/rbenv-installer | bash
  
  # add ~/.rbenv/bin to your $PATH so that you can use the rbenv command line utility
  echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bashrc

  # make rbenv load automatically by adding it to .bashrc
  echo 'eval "$(rbenv init -)"' >> ~/.bashrc
  
  # apply changes to current shell session
  source ~/.bashrc
  
  # verify if rbenv is installed[Output - rbenv is a function]
  type rbenv

  # install ruby-build plugin from official github repo, adds the rbenv install command
  git clone https://github.com/rbenv/ruby-build.git
  
  # Create this temporary environment variable and run the script[requires sudo]
  PREFIX=/usr/local sudo ./ruby-build/install.sh
  
  # list out ruby versions available for install [choose any version > 2.5.0]
  rbenv install -l
  
  # install a version from the list
  rbenv install [VERSION_SELECTED]
  # for example
  rbenv install 2.7.6
  
  # set selected version as default
  rbenv global [VERSION_SELECTED]
  ```
- Installing Dependencies **Windows**
  - Please use [WSL](https://docs.microsoft.com/en-us/windows/wsl/)
  - Read the [documentation](https://docs.microsoft.com/en-us/windows/wsl/install) on installing WSL on Windows
  - Once the WSL is set up, use the same steps as above in ubuntu/debian to install dependencies

- Installing Dependencies in **MacOS**
  ```bash
  # install dependencies
  brew install rsync openssl rbenv ruby-build ruby_dev

  # install ruby 2.7.5
  rbenv install 2.7.5

  # initialize rbenv
  eval "$(rbenv init -)"

  # set the ruby version locally to 2.7.5
  rbenv local 2.7.5
  ```
  
- Check if dependencies are installed correctly (each should give a version as output)
  ```bash
  ruby -v
  gem -v
  ```
  
### Setting up the repository
  
- To get the site up and running locally, follow the below steps:

  **Note** You need to have a full Bash environment. If you're on Windows, please use WSL.

- Fork the repository
  You can get your own fork/copy of [weaviate.io](https://github.com/semi-technologies/weaviate-io) by using the `Fork` button

- Create a local clone of the website:
  ```
  git clone git@github.com:[YOUR-USERNAME]/weaviate-io.git
  ```
- Change into the weaviate-io directory
  ```
  cd weaviate-io
  ```
- Add upstream URL, this acts as a reference from original weaviate.io's repository
  ```bash
  git remote add upstream git@github.com:semi-technologies/weaviate-io.git
  ```
- Install Jekyll and Bundler
  ```bash
  gem install jekyll bundler
  ```
- Verify jekyll installation
  ```bash
  jekyll -v
  ```
- Perform the following commands to install dependencies and structure the website properly:
  ```
  bundle install
  ```
- Build the site and make it available on your local server
  ```
  bundle exec jekyll serve
  ```
- Browse [http://localhost:4000](http://localhost:4000) to view the website.

### OG image generator

OG images for documentation are created and linked automatically. The `og` tag in 
front matter should only be used for manual override. E.g. The `architecture/binary_passage_retrieval.md` image
would be located at `img/og/og-documentation/architecture-binary-passage-retrieval.jpg`.

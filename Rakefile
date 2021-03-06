#############################################################################
#
# Modified version of jekyllrb Rakefile
# https://github.com/jekyll/jekyll/blob/master/Rakefile
#
#############################################################################

require 'rake'
require 'date'
require 'yaml'
# require 'html/proofer'

CONFIG = YAML.load(File.read('_config.yml'))
USERNAME = CONFIG["username"]
REPO = CONFIG["repo"]
SOURCE_REPO = CONFIG["source_url"]
SOURCE_BRANCH = CONFIG["branch"]
DESTINATION_BRANCH = "gh-pages"
CNAME = CONFIG["CNAME"]

def check_destination
  unless Dir.exist? CONFIG["destination"]
    # sh "git clone https://$GIT_NAME:$GH_TOKEN@github.com/#{USERNAME}/#{REPO}.git #{CONFIG["destination"]}"
    sh "git clone https://$GIT_NAME:$GH_TOKEN@#{SOURCE_REPO} #{CONFIG["destination"]}"
  end
end

namespace :site do
  # desc "build and test website"
  # task :test do
  #   check_destination
  #   sh "bundle exec jekyll build"
  #   HTML::Proofer.new("_site", {:href_ignore=> ['http://localhost:4000'], :verbose => true}).run
  # end

  desc "Generate the site"
  task :build do
    check_destination
    sh "bundle exec jekyll build"
  end

  desc "Generate the site and serve locally"
  task :serve do
    check_destination
    sh "bundle exec jekyll serve"
  end

  desc "Generate the site, serve locally and watch for changes"
  task :watch do
    sh "bundle exec jekyll serve --watch"
  end

  desc "Generate the site and push changes to remote origin"
  task :deploy do
    # Configure git if this is run in CI
    if ENV["CI"]
      sh "git config --global user.name $GIT_NAME"
      sh "git config --global user.email $GIT_EMAIL"
      sh "git config --global push.default simple"
    end

    # Make sure destination folder exists as git repo
    check_destination

    sh "git checkout #{SOURCE_BRANCH}"
    Dir.chdir(CONFIG["destination"]) { sh "git checkout #{DESTINATION_BRANCH}" }

    unless ENV["CI"]
      # Generate the site
      sh "bundle exec jekyll build"
    end

    # Commit and push to github
    sha = `git log`.match(/[a-z0-9]{40}/)[0]
    Dir.chdir(CONFIG["destination"]) do
      # check if there is anything to add and commit, and pushes it
      sh "if [ -n '$(git status)' ]; then
            echo '#{CNAME}' > ./CNAME;
            git add --all .;
            git commit -m 'Updating to #{USERNAME}/#{REPO}@#{sha}.';
            git push --quiet origin #{DESTINATION_BRANCH};
         fi"
      puts "Pushed updated branch #{DESTINATION_BRANCH} to GitHub Pages"
    end
  end
end

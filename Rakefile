require "tmpdir"


GITHUB_REPONAME = "Desgard/algo"
GITHUB_REPO_BRANCH = "gh-pages"
DEST = "public/."
IMG_DEST = "static/."

task default: %w[publish]

desc "Generate book static files by Hugo"
task :gen do
  system "hugo"
end

desc "Run Hugo server to test"
task :s do 
  system "hugo server -D"
end

desc "Copy image resource to img branch"
task :img do
  Dir.mktmpdir do |tmp|
    cp_r IMG_DEST, tmp
    pwd = Dir.pwd
    Dir.chdir tmp
    
    system "git init"
    system "git checkout --orphan img"
    system "git add ."
    system "git config user.email 'gua@desgard.com'"
    msg = "#{Time.now.utc} - Updated"
    system "git commit -am #{msg.inspect}"
    system "git remote add origin git@github.com:#{GITHUB_REPONAME}.git"
    system "git push origin img --force"

    Dir.chdir pwd
  end
end

desc "Generate and publish book to my Repo"
task :publish => [:gen] do
  Dir.mktmpdir do |tmp|
    cp_r DEST, tmp
    pwd = Dir.pwd
    Dir.chdir tmp
    
    system "git init"
    system "git checkout --orphan #{GITHUB_REPO_BRANCH}"    
    system "git add ."
    system "git config user.email 'gua@desgard.com'"
    msg = "#{Time.now.utc}"
    system "git commit -am '勤劳的冬瓜在 #{msg} 又更新了 algo 算法小册，快来阅读吧！👉🏻 www.desgard.com/algo'"
    system "git remote add origin git@github.com:#{GITHUB_REPONAME}.git"
    system "git push origin #{GITHUB_REPO_BRANCH} --force"

    Dir.chdir pwd
  end
end
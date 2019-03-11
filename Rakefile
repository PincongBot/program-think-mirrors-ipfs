require 'rake'
require 'date'

def check_destination_blog
  if Dir.exist? "blog"
    Dir.chdir("blog") { sh "git pull" }
  else
    sh "git clone --depth=1 https://github.com/program-think-mirrors/blog.git blog"
  end
  sh "mkdir programthink"
  sh "mv blog/html programthink/blog"
end

def check_destination_books
  if Dir.exist? "books"
    Dir.chdir("books") { sh "git pull" }
  else
    sh "git clone --depth=1 https://github.com/program-think-mirrors/books.git books"
  end
end

task :init do

    unless Dir.exist? "#{ENV['HOME']}/.ipfs/keystore"
      sh "ipfs init"
    end

end

task :deploy do

    # Detect pull request
    if ENV['TRAVIS_PULL_REQUEST'].to_s.to_i > 0
      puts 'Pull request detected.'
      exit
    end

    puts DateTime.now

    # blog
    fork do
      puts "cloning blog"
      check_destination_blog
    end

    # books
    fork do
      puts "cloning books"
      check_destination_books
    end

    puts DateTime.now

    Process.waitall

    fork do
      i = 0
      while true do
        if i % 30 == 0 then
          peers = `ipfs swarm peers | wc -l`.match(/\d+/)[0].to_i
          puts "Connected to #{peers} peers"  
        end
  
        sleep(1)
        i += 1
      end
    end

    puts "'mv books programthink/'"
    sh "mv books programthink/"
    puts "done"

    sh "ipfs swarm peers"
    ipfs_hash = `ipfs add -r -Q programthink`.match(/\w+/)[0]
    sh "ipfs pin add -r /ipfs/#{ipfs_hash}"
    sh "ipfs name publish --key=key #{ipfs_hash}"

    exit

end

task :sync, [:minutes] do |t, args|

    args.with_defaults(:minutes => 10)
    minutes = args.minutes.to_i

    max = (minutes - 1) * 60 + 40
    i = 0

    puts "\nsyncing"

    while i < max do

      if i % 30 == 0 then
        peers = `ipfs swarm peers | wc -l`.match(/\d+/)[0].to_i
        puts "Connected to #{peers} peers"  
      end

      sleep(1)
      i += 1

    end
  
end

task :test do

    puts "\ntesting"

    ipns_hash = `ipfs key list -l`.match(/(\w{46}) key/)[1]
    ipfs_hash = `ipfs name resolve #{ipfs_hash}`.match(/\w{46}/)[0]

    gateways = [ "cloudflare-ipfs.com", "ipfs.io", "siderus.io" ]

    gateways.each do |i|
      sh "curl 'https://#{i}/ipfs/#{ipfs_hash}' > /dev/null"
      sh "curl 'https://#{i}/ipns/#{ipns_hash}' > /dev/null"
    end

end

task :timing_output do

    i = 0

    while true do

      if i % 30 == 0 then
        puts ""
        sh "du -h -s /home/travis/"
      end

      sleep(1)
      i += 1

    end

end

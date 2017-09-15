# Refer to https://github.com/plusjade/jekyll-bootstrap/blob/master/Rakefile

require "rubygems"
require 'rake'
require 'yaml'
require 'time'

SOURCE = "."
CONFIG = {
  'posts' => File.join(SOURCE, "_posts"),
  'works' => File.join(SOURCE, "_works"), 
  'post_ext' => "md",
}

# Usage: rake work title="A Title" [date="2012-02-09"] [tags=[tag1,tag2]]
desc "Begin a new post in #{CONFIG['works']}"
task :work do
  abort("rake aborted: '#{CONFIG['works']}' directory not found.") unless FileTest.directory?(CONFIG['works'])
  title = ENV["title"] || "new-work"
  tags = ENV["tags"] || "[]"
  slug = title.downcase.strip.gsub(' ', '-').gsub(/[^\w-]/, '')
  begin
    date = (ENV['date'] ? Time.parse(ENV['date']) : Time.now).strftime('%Y-%m-%d')
  rescue => e
    puts "Error - date format must be YYYY-MM-DD, please check you typed it correctly!"
    exit -1
  end
  filename = File.join(CONFIG['works'], "#{date}-#{slug}.#{CONFIG['post_ext']}")
  if File.exist?(filename)
    abort("rake aborted!") if ask("#{filename} already exists. Do you want to overwrite?", ['y', 'n']) == 'n'
  end

  puts "Creating new works: #{filename}"
  open(filename, 'w') do |work|
    work.puts "---"
    work.puts "layout: post"
    work.puts "title: \"#{title.gsub(/-/,' ')}\""
    work.puts 'description: ""'
    work.puts "date: #{date}"
    work.puts "tags: #{tags}"
    work.puts "comments: true"
    work.puts "---"
  end
end # task :work

desc "Install Jekyll Plugins"
task :geminstall do
  system "sudo gem install jekyll-seo-tag jekyll-paginate jekyll-admin"
end # task :geminstall

desc "Launch preview environment"
task :preview do
  system "jekyll serve --incremental"
end # task :preview

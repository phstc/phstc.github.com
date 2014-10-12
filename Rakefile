require 'fileutils'


namespace :draft do
  # Based on http://albertogrespan.com/blog/rake-tasks-and-jekyll-posts/
  desc 'Creating a new draft for post/entry'
  task :new do
    puts "What's the name for your next post?"

    name = STDIN.gets.chomp
    slug = "#{name}"
    slug = slug.tr('ÁáÉéÍíÓóÚú', 'AaEeIiOoUu')
    slug = slug.downcase.strip.gsub(' ', '-')

    FileUtils.touch("_drafts/#{slug}.md")

    open("_drafts/#{slug}.md", 'a') do |file|
      file.puts '---'
      file.puts 'layout: post'
      file.puts "title: #{name}"
      file.puts '---'
    end
  end

  desc "Copy draft to production post"
  task :ready do
    puts "What's the name for your next post?"

    puts 'Posts in _drafts:'
    files = Dir.foreach('_drafts').each_with_index.map do |file_name, i|
      next if %w[. .. .keep].include? file_name

      puts "#{i} - #{file_name}"

      file_name
    end

    puts "What's the name of the draft to post?"

    post_index = STDIN.gets.chomp
    post_name = files[post_index.to_i]
    post_date = Time.now.strftime('%F')

    FileUtils.mv("_drafts/#{post_name}",  "_posts/#{post_name}")
    FileUtils.mv("_posts/#{post_name}",   "_posts/#{post_date}-#{post_name}")

    puts 'Post copied and ready to deploy!'
  end

end

desc 'Start jekyll in development mode'
task :server do
  exec 'jekyll server --watch --trace --drafts'
end
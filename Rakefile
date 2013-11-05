require 'rake'
require 'jekyll'

task :default => :wordcount

task :wordcount do
  config = Jekyll.configuration show_drafts: true
  site   = Jekyll::Site.new(config)
  site.read

  word_count = site.posts.inject(0) { |sum, post| sum + post.content.split.length }

  # The planning post was written before NaNoWriMo, so it doesn't count!
  planning_post_word_count = 1_882
  word_count              -= planning_post_word_count

  goal                  = 50_000
  fraction_of_the_month = Rational(Date.today.day, 30)
  target_so_far         = fraction_of_the_month * goal

  puts "Total words so far: #{word_count}."

  shortfall = (target_so_far - word_count).to_i
  if shortfall > 0
    puts "You only need to write another #{shortfall} words today to get back on track."
  else
    puts "Well done, you're ahead of the game by #{shortfall.abs} words!"
  end
end

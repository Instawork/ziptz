require 'bundler/setup'
Bundler.setup(:default, :development)

require 'rspec/core/rake_task'
RSpec::Core::RakeTask.new :spec do |t|
  t.rspec_opts = %w[--color]
end

RSpec::Core::RakeTask.new :specdoc do |t|
  t.rspec_opts = %w[-fl]
end

task default: :spec

desc 'Open an irb session preloaded with this library'
task :console do
  sh 'irb -rubygems -I lib -r ziptz.rb'
end

desc 'Create ziptz db from zipcodes database'
task :create_ziptz do
  require 'yaml'
  require 'active_record'

  db_config = YAML.load(File.open('database.yml'))
  ActiveRecord::Base.establish_connection(db_config)

  class ZipCode < ActiveRecord::Base
    self.table_name = 'ZIPCodes'
    self.primary_key = 'ZipCode'

    alias_attribute :zip_code, :ZipCode
    alias_attribute :time_zone, :TimeZone
    alias_attribute :day_light_saving, :DayLightSaving
  end

  puts 'Retrieving zip codes from database'

  data = {}
  ZipCode.find_each do |zip|
    data[zip.zip_code] ||= {}
    data[zip.zip_code][:tz] ||= zip.time_zone
    data[zip.zip_code][:dst] ||= zip.day_light_saving
  end

  puts 'Writing tz.data'

  lines = data.map { |k, v| "#{k}=#{v[:tz]}" }
  lines.sort!

  File.open('data/tz.data', 'w') do |f|
    lines.each { |line| f.puts line }
  end

  puts 'Writing dst.data'
  lines = data.map { |k, v| "#{k}=#{v[:dst] =~ /y/i ? 1 : 0}" }
  lines.sort!

  File.open('data/dst.data', 'w') do |f|
    lines.each { |line| f.puts line }
  end
end

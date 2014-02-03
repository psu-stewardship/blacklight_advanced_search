require 'rake'
require 'rake/testtask'
require 'rdoc/task'

require 'bundler/setup'
Bundler::GemHelper.install_tasks

require 'rspec/core/rake_task'

require 'blacklight'
require 'blacklight_marc'
spec = Gem::Specification.find_by_name("blacklight_marc")
marc_root = spec.gem_dir
import File.join(marc_root, 'lib', 'railties', 'solr_marc.rake')
Rake::Task.define_task(:environment)

task :default => :spec

desc "Run specs"
RSpec::Core::RakeTask.new do |t|

end


desc "Execute Continuous Integration build"
task :ci do

  require 'combustion'
  require 'blacklight'
Combustion.initialize!
  unless ENV['environment'] == 'test'
    exec("rake ci environment=test") 
  end

  require 'rails/generators'
  require File.join(Blacklight.root, 'lib', 'generators', 'blacklight', 'jetty_generator.rb')

  Blacklight::Jetty.start(["--save_location=jetty", "--force"])

  ENV['RAILS_ENV'] = 'test'
  ENV['CONFIG_PATH'] = File.expand_path(File.join(marc_root, 'lib', 'generators', 'blacklight_marc', 'templates', 'config', 'SolrMarc', 'config-test.properties'))
  ENV['SOLRMARC_JAR_PATH'] = File.expand_path(File.join(marc_root, 'lib', 'SolrMarc.jar'))
  ENV['SOLR_PATH'] = File.expand_path(File.join('jetty', 'solr'))
  ENV['SOLR_WAR_PATH'] = File.expand_path(File.join('jetty', 'webapps', 'solr.war'))
  Rake::Task['solr:marc:index_test_data'].invoke


  require 'jettywrapper'
  jetty_params = {
    :jetty_home => File.expand_path(File.dirname(__FILE__) + '/jetty'),
    :quiet => false,
    :jetty_port => 8888,
    :solr_home => File.expand_path(File.dirname(__FILE__) + '/jetty/solr'),
    :startup_wait => 30
  }

  error = Jettywrapper.wrap(jetty_params) do
    Rake::Task['spec'].invoke
  end
  raise "test failures: #{error}" if error
end

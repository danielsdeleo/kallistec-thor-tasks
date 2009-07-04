# encoding: UTF-8

require "fileutils"
require "logger"
require "rubygems"
require "thor"

module ProjectGenerator
  
  ::String.class_eval do
    def classify
      split("_").map { |w| w.capitalize }.join("")
    end
  end
  
  MAGIG_UTF8 = "# encoding: UTF-8"
  
  RAKEFILE=<<RAKE
# encoding: UTF-8
require "spec/rake/spectask"
require "cucumber"
require "cucumber/rake/task"

task :default => :spec

desc "Run Cucumber Features"
Cucumber::Rake::Task.new do |t|
  t.cucumber_opts = "-c -n"
end

desc "Run all of the specs"
Spec::Rake::SpecTask.new do |t|
  t.spec_opts = ['--options', \"spec/spec.opts\"]
  t.fail_on_error = false
end

namespace :spec do

  desc "Generate HTML report for failing examples"
  Spec::Rake::SpecTask.new('report') do |t|
    t.spec_files = FileList['failing_examples/**/*.rb']
    t.spec_opts = ["--format", "html:doc/tools/reports/failing_examples.html", "--diff", '--options', '"spec/spec.opts"']
    t.fail_on_error = false
  end

  desc "Run all spec with RCov" 
  Spec::Rake::SpecTask.new(:rcov) do |t|
    t.rcov = true
    t.rcov_dir = 'doc/tools/coverage/'
    t.rcov_opts = ['--exclude', 'spec']
  end

end
RAKE

MOCK_WITH_MOCHA=<<MOCHA
Spec::Runner.configure do |config|
  config.mock_with :mocha
end
MOCHA

  def create_tree(name)
    log("creating project tree for #{name}")
    mkdir_p("spec/unit")
    mkdir_p("lib/" + name)
    mkdir_p("features/steps")
  end
  
  def spec_opts(fh)
    log("creating spec.opts")
    fh.puts "-f specdoc -c -t 2"
  end
  
  def rakefile(fh)
    log("creating rakefile")
    fh.puts RAKEFILE
  end
  
  def lib_root_file(fh, name)
    log("creating ``root'' library file")
    fh.puts MAGIG_UTF8
    fh.puts "unless defined?(#{project_root_const})"
    fh.puts "  " + project_root_const + " = File.dirname(__FILE__) + '/'"
    fh.puts "end"
    fh.puts
  end
  
  def spec_helper(fh, name)
    log("generating spec/spec_helper.rb")
    fh.puts MAGIG_UTF8
    fh.puts "require 'rubygems'"
    fh.puts
    fh.puts "require File.dirname(__FILE__) + '/../lib/#{name}.rb'"
    fh.puts 
    fh.puts MOCK_WITH_MOCHA
    fh.puts
    fh.puts "include #{name.classify}"
    fh.puts
  end
  
  def git_init
    log("initializing git repo")
    system("git init")
  end
  
  def git_initial_commit
    log("performing initial commit")
    system("git add .")
    system("git commit -a -m \"initial commit\"")
  end
  
  def log(msg)
    unless @logger 
      @logger = Logger.new(STDERR)
      @logger.formatter = SimpleLogs.new
      @logger.level = Logger::DEBUG
    end
    @logger.debug(msg)
  end
  
  def spec_stub(fh, name)
    fh.puts MAGIG_UTF8
    fh.puts "require File.dirname(__FILE__) + \"/../spec_helper\""
    fh.puts
    fh.puts "describe #{name.classify} do"
    fh.puts
    fh.puts "end"
  end
  
  def project_name
     Dir.glob("lib/*.rb").first.split("/").last.sub(/\.rb$/, '')
  end
  
  def class_stub(fh, name)
    fh.puts MAGIG_UTF8
    fh.puts
    fh.puts "module #{project_name.classify}"
    fh.puts "  class #{name.classify}"
    fh.puts "    "
    fh.puts "  end"
    fh.puts "end"
  end
  
  def project_root_const
    project_name.upcase + "_ROOT"
  end
  
  def add_require(fh, name)
    log("updating project initializer to load lib/#{project_name}/#{name}.rb")
    fh.puts "require #{project_root_const} + \"/#{project_name}/#{name}\""
  end
  
  class SimpleLogs < Logger::Formatter
    def call(severity, time, program_name, message)
      "[#{severity}] #{message} \n"
    end
  end
  
end


class Gen < Thor
  include FileUtils
  include ProjectGenerator
  
  desc "new_rb", "create a new ruby project"
  def new_rb(name)
    if File.exist?(File.expand_path "~/ruby/#{name}")
      puts ("[FAILED] Ruby project #{name} already exists")
      exit 127
    end
    
    Dir.chdir("~/ruby") unless Dir.pwd == File.expand_path("~/ruby")
    log("in working dir #{Dir.pwd}")
    mkdir(name)
    Dir.chdir(name)
    log("in working dir #{Dir.pwd}")
    init_rb(name)
  end
  
  desc "init_rb", "initialize a new ruby project"
  def init_rb(name)
    create_tree(name)
    File.open("spec/spec.opts", "w+") do |file|
      spec_opts(file)
    end
    File.open("Rakefile", "w+") do |file|
      rakefile(file)
    end
    File.open("lib/#{name}.rb", "w+") do |file|
      lib_root_file(file, name)
    end
    File.open("spec/spec_helper.rb", "w+") do |f|
      spec_helper(f, name)
    end
    git_init
    touch("README.rdoc")
    git_initial_commit
    log("created project #{name} in #{File.expand_path("~/ruby/" + name)}")
  end
  
  def class(name)
    spec_file = "spec/unit/#{name}_spec.rb"
    lib_file = "lib/#{project_name}/#{name}.rb"
    if File.exist?(spec_file) || File.exist?(lib_file)
      puts "[FAILED] one or both of #{spec_file}, #{lib_file} exists."
      exit 127
    end
    log("creating stub class file in #{spec_file}")
    File.open(spec_file, "w+") do |f|
      spec_stub(f, name)
    end
    log("creating stub class file in #{lib_file}")
    File.open(lib_file, "w+") do |f|
      class_stub(f, name)
    end
    File.open("lib/#{project_name}.rb", "a+") do |f|
      add_require(f, name)
    end
  end
  
end

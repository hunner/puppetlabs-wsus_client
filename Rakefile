require 'rake'
require 'rspec/core/rake_task'
require 'puppetlabs_spec_helper/rake_tasks'
require 'puppet'
require 'rubygems'
require 'puppet-lint/tasks/puppet-lint'
require 'puppet_blacksmith/rake_tasks' if Bundler.rubygems.find_name('puppet-blacksmith').any?

begin
  require 'beaker/tasks/test' unless RUBY_PLATFORM =~ /win32/
rescue LoadError
  #Do nothing, only installed with system_tests group
end


# If puppet does not support symlinks (e.g., puppet <= 3.5) we cannot use
# puppetlabs_spec_helper's `rake spec` task because it requires symlink
# support. Redefine `rake spec` to avoid calling `rake spec_prep` (requires
# symlinks to place fixtures) and restrict the pattern match only files under
# the 'unit' directory (tests in other dirs require fixtures).
if Puppet::Util::Platform.windows? and !Puppet::FileSystem.respond_to?(:symlink)
  ENV["SPEC"] = "./spec/{unit,integration}/**/*_spec.rb"
  Rake::Task[:spec].clear if Rake::Task.task_defined?(:spec)
  task :spec do
    Rake::Task[:spec_standalone].invoke
    Rake::Task[:spec_clean].invoke
  end
end

# These lint exclusions are in puppetlabs_spec_helper but needs a version above 0.10.3
# Line length test is 80 chars in puppet-lint 1.1.0
PuppetLint.configuration.send('disable_80chars')
# Line length test is 140 chars in puppet-lint 2.x
PuppetLint.configuration.send('disable_140chars')

desc "Validate manifests, templates, and ruby files"
task :validate do
  Dir['manifests/**/*.pp'].each do |manifest|
    sh "puppet parser validate --noop #{manifest}"
  end
  Dir['spec/**/*.rb','lib/**/*.rb'].each do |ruby_file|
    sh "ruby -c #{ruby_file}" unless ruby_file =~ /spec\/fixtures/
  end
  Dir['templates/**/*.erb'].each do |template|
    sh "erb -P -x -T '-' #{template} | ruby -c"
  end
end

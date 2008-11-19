require 'rubygems'
require 'rake/gempackagetask'
require 'rubygems/specification'
require 'date'
require 'fileutils'
require 'rbconfig'

GEM = "ffi"
GEM_VERSION = "0.2.0"
AUTHOR = "Wayne Meissner"
EMAIL = "wmeissner@gmail.com"
HOMEPAGE = "http://kenai.com/projects/ruby-ffi"
SUMMARY = "A Ruby foreign function interface (compatible with Rubinius and JRuby FFI)"
LIBEXT = Config::CONFIG['host_os'].downcase =~ /darwin/ ? "dylib" : "so"
GMAKE = Config::CONFIG['host_os'].downcase =~ /bsd/ ? "gmake" : "make"
LIBTEST = "build/libtest.#{LIBEXT}"

spec = Gem::Specification.new do |s|
  s.name = GEM
  s.version = GEM_VERSION
  s.platform = Gem::Platform::RUBY
  s.has_rdoc = true
  s.extra_rdoc_files = ["README", "LICENSE"]
  s.summary = SUMMARY
  s.description = s.summary
  s.author = AUTHOR
  s.email = EMAIL
  s.homepage = HOMEPAGE
  s.rubyforge_project = 'ffi' 
  s.extensions = %w(ext/extconf.rb gen/Rakefile)
  
  s.require_path = 'lib'
  s.autorequire = GEM
  s.files = %w(LICENSE README Rakefile) + Dir.glob("{ext,lib,nbproject,samples,specs}/**/*")
end

if RUBY_PLATFORM == "java"
  desc "Run all specs"
  task :specs do
    sh %{#{Gem.ruby} -S spec #{Dir["specs/**/*_spec.rb"].join(" ")} -fs --color}
  end
  desc "Run rubinius specs"
  task :rbxspecs do
    sh %{#{Gem.ruby} -S spec #{Dir["specs/rbx/**/*_spec.rb"].join(" ")} -fs --color}
  end
else
  desc "Run all specs"
  task :specs do
    ENV["MRI_FFI"] = "1"
    sh %{#{Gem.ruby} -Ibuild -Ilib -S spec #{Dir["specs/**/*_spec.rb"].join(" ")} -fs --color}
  end
  desc "Run rubinius specs"
  task :rbxspecs do
    ENV["MRI_FFI"] = "1"
    sh %{#{Gem.ruby} -Ibuild -Ilib -S spec #{Dir["specs/rbx/**/*_spec.rb"].join(" ")} -fs --color}
  end
end

Rake::GemPackageTask.new(spec) do |pkg|
  pkg.gem_spec = spec
end

desc "install the gem locally"
task :install => [:package] do
  sh %{sudo #{Gem.ruby} -S gem install pkg/#{GEM}-#{GEM_VERSION}}
end

desc "create a gemspec file"
task :make_spec do
  File.open("#{GEM}.gemspec", "w") do |file|
    file.puts spec.to_ruby
  end
end
file "build/Makefile" do
  FileUtils.mkdir_p("build") unless File.directory?("build")
  sh %{cd build && #{Gem.ruby} ../ext/extconf.rb}
end
desc "Compile the native module"
task :compile => "build/Makefile" do
  sh %{cd build; make}
end
desc "Clean all built files"
task :clean do
  sh %{cd build;make distclean} if File.exists?("build/Makefile")
  FileUtils.rm_rf("build")
  FileUtils.rm_rf("conftest.dSYM")
  FileUtils.rm_f(Dir["pkg/*.gem"])
  FileUtils.rm_f("Makefile")
end
task "build/libtest.#{LIBEXT}" do
  sh %{#{GMAKE} -f libtest/GNUmakefile}
end
desc "Test the extension"
task :test => [ :compile, LIBTEST, :specs ] do

end
namespace :bench do
  ITER = ENV['ITER'] ? ENV['ITER'].to_i : 100000
  Dir["bench/bench_*.rb"].each do |bench|
    task File.basename(bench, ".rb")[6..-1] => [ LIBTEST, :compile ] do
      sh %{#{Gem.ruby} -Ibuild -Ilib #{bench} #{ITER}}
    end
  end
  task :all => LIBTEST do
    Dir["bench/bench_*.rb"].each do |bench|
      sh %{#{Gem.ruby} -Ibuild -Ilib #{bench}}
    end
  end
end
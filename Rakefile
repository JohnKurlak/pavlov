require 'rubygems'
require 'net/http'
require 'rake/clean'
require 'packr'
require 'zip/zip'
require 'find'
require 'fileutils'
include FileUtils

task :default => "test:adapter:qunit"

# list of browsers to auto-bind to JsTestDrive Server
# non-existent browsers will be ignored
BROWSERS = [
  '/Applications/Safari.app/Contents/MacOS/Safari',
  '/Applications/Firefox.app/Contents/MacOS/firefox',
  '/Applications/Chromium.app/Contents/MacOS/Chromium',
  '/Applications/Opera.app/Contents/MacOS/Opera',
  'C:/Program Files/Mozilla Firefox/firefox.exe',
  'C:/Program Files/Internet Explorer/iexplore.exe',
  'C:/Program Files/Safari/Safari.exe',
  'C:/Program Files/Opera/opera.exe' ]


desc "'Compiles' sources and examples together"
task :build => [:clean] do
  # build dist and lib directories
  mkdir 'dist'
  mkdir 'dist/lib'
  mkdir 'dist/example'

  # copy src
  cp 'pavlov.js', 'dist/pavlov.js'
  cp 'pavlov.qunit.js', 'dist/pavlov.qunit.js'
  cp 'pavlov.yui3.js', 'dist/pavlov.yui3.js'

  # copy documentation
  cp 'README.markdown', 'dist/README.markdown'

  # copy lib
  cp 'lib/qunit.js', 'dist/lib/qunit.js'
  cp 'lib/qunit.css', 'dist/lib/qunit.css'
  cp 'spec/lib/jquery/GPL-LICENSE.txt', 'dist/lib/GPL-LICENSE.txt'
  cp 'spec/lib/jquery/MIT-LICENSE.txt', 'dist/lib/MIT-LICENSE.txt'

  # copy example
  cp 'example/example.specs.qunit.html', 'dist/example/example.specs.qunit.html'
  cp 'example/example.specs.yui3.html', 'dist/example/example.specs.yui3.html'
  cp 'example/example.specs.js', 'dist/example/example.specs.js'

  # minify 
  minify('dist/pavlov.js')  
  minify('dist/pavlov.qunit.js')
  minify('dist/pavlov.yui3.js')
end

def minify(file)
  # minify src
  source = File.read(file)
  minified = Packr.pack(source, :shrink_vars => true, :base62 => false)
  header = /\/\*.*?\*\//m.match(source)

  # inject header
  File.open(file.slice(0, file.length - 3) + '.min.js', 'w') do |combined|
    combined.puts(header)
    combined.write(minified)
  end
end

desc "Generates a releasable zip archive"
task :release => [:build] do
  root = pwd+'/dist'
  zip_archive = pwd+'/dist/Pavlov.zip'

  Zip::ZipFile.open(zip_archive, Zip::ZipFile::CREATE) do |zip|
    Find.find(root) do |path|
      Find.prune if File.basename(path)[0] == ?.
      dest = /dist\/(\w.*)/.match(path)
      zip.add(dest[1],path) if dest
    end
  end
end

namespace :test do 
  namespace :adapter do
    
    desc "Run the qunit tests in default browser"
    task :qunit => [:build] do
      begin
        # mac
        sh("open spec/pavlov.specs.qunit.html")
      rescue
        # windows
        sh("start spec/pavlov.specs.qunit.html")
      end
    end

    desc "Run the yui3 tests in default browser"
    task :yui3 => [:build] do
      begin
        # mac
        sh("open spec/pavlov.specs.yui3.html")
      rescue
        # windows
        sh("start spec/pavlov.specs.yui3.html")
      end
    end
    
  end
end

desc "Run the tests against JsTestDriver"
task :testdrive => [:build] do
  sh("java -jar spec/lib/js-test-driver/JsTestDriver.jar --tests all --captureConsole --reset")
end


desc "Start the JsTestDriver server"
task :server => [:install_server] do
  browsers = BROWSERS.find_all{|b| File.exists? b}.join(',')
  sh("java -jar spec/lib/js-test-driver/JsTestDriver.jar --port 9876 --browser \"#{browsers}\"")
end


desc "Download Google JsTestDriver"
task :install_server do
  if !File.exist?('spec/lib/js-test-driver/JsTestDriver.jar') then
    puts 'Downloading JsTestDriver from Google (http://js-test-driver.googlecode.com/files/JsTestDriver-1.0b.jar) ...'
    Net::HTTP.start("js-test-driver.googlecode.com") do |http|
      resp = http.get("/files/JsTestDriver-1.0b.jar")
      open("spec/lib/js-test-driver/JsTestDriver.jar", "wb") do |file|
        file.write(resp.body)
      end
    end
    puts 'JsTestDriver Downloaded'
  end
end


# clean deletes built copies
CLEAN.include('dist/')
# clobber cleans and uninstalls JsTestDriver server
CLOBBER.include('spec/lib/js-test-driver/*.jar')

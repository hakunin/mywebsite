begin
  require 'ant'
rescue LoadError
  puts 'This Rakefile requires JRuby. Please use jruby -S rake.'
  exit 1
end

require 'rake/clean'
require 'dubious_tasks'

OUTDIR = 'WEB-INF/classes'
CLEAN.include(OUTDIR)
CLOBBER.include('WEB-INF/appengine-generated')

task :set_compile_options do |t|
  def t.needed?;false;end
  
  mirah_compile_options :dest_path => OUTDIR,
                        :source_paths => [File.expand_path('lib'), File.expand_path('app') ],
                        :compiler_options => ['--classpath', [File.expand_path(OUTDIR), *FileList["WEB-INF/lib/*.jar"].map{|f|File.expand_path(f)}].join(':') + ':' + CLASSPATH ]
end

def class_files_for files
  files.map do |f|
    explode = f.split('/')[1..-1]
    explode.last.gsub!(/(^[a-z]|_[a-z])/) {|m|m.sub('_','').upcase}
    explode.last.sub! /\.(duby|java|mirah)$/, '.class'
    OUTDIR + '/' + explode.join('/')
  end
end

MODEL_JAR = "WEB-INF/lib/mirahdatastore.jar"

LIB_MIRAH_SRC = Dir["lib/**/*.{duby,mirah}"]
LIB_JAVA_SRC  = Dir["lib/**/*.java"]
LIB_SRC = LIB_MIRAH_SRC + LIB_JAVA_SRC

APP_SRC = Dir["app/**/{*.duby,*.mirah}"]
TEMPLATES = Dir["app/views/**/*.erb"]

LIB_CLASSES = class_files_for LIB_SRC
APP_CLASSES = class_files_for APP_SRC
APP_MODEL_CLASSES = APP_CLASSES.select {|app| app.include? '/models' }
APP_CONTROLLER_CLASSES = APP_CLASSES.select {|app| app.include? '/controllers' }
APP_APPLICATION_CONTROLLER_CLASS = APP_CONTROLLER_CLASSES.find {|controller| controller.include? 'ApplicationController' }

directory OUTDIR



(APP_CLASSES+LIB_CLASSES).zip(APP_SRC+LIB_SRC).each do |klass,src|
  file klass => src
end




filemap = {}
classmap = {}


APP_SRC.each { |file|
  name = File.basename(file)
  name = name.split('_').map{ |f| f.capitalize }.join('').split('.')[0]
  dest_name = name+'.class'
  #puts "++++"+name
  compiled = OUTDIR + '/' + File.dirname(file).gsub('app/', '') + '/' + dest_name
  filemap[name] = { :src => file, :dest => compiled }
  classmap[compiled] = { :src => file, :class => name }
}

dependencies = {}

APP_CLASSES.each { |f|
  #puts f
  #p classmap[f]
  
  dependencies[f] = {}
  
  if (!classmap[f])
    warn "WARNING! info for #{f} not found"
    exit
  end
  if f.include? 'html.erb'
    next
  end
  
  
  File.open(classmap[f][:src]).each { |line|
    line.scan(/[A-Z][A-Za-z0-9]*/).each { |klass|
      if (src = filemap[klass])
        if (klass != classmap[f][:class])
          if !dependencies[f][src[:dest]]
            #puts f + " depends on " + src[:dest]
            dependencies[f][src[:dest]] = true
          end
        end
      else
        #puts "  "+klass
      end
    }
  }
  file f => dependencies[f].keys
}



APP_CLASSES.each do |f|
  file f => LIB_CLASSES
end

file MODEL_JAR => MODEL_SRC_JAR do |t|
  cp MODEL_SRC_JAR, MODEL_JAR
end

appengine_app :app, '', '.' => [:set_compile_options] + APP_CLASSES + LIB_CLASSES

namespace :compile do
  task :app => APP_CLASSES

  task :java => OUTDIR do
    ant.javac :srcdir => 'lib', :destdir => OUTDIR, :classpath => CLASSPATH
  end
end

desc "compile app"
task :compile => 'compile:app'


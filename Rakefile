require 'rake/clean'

VERSION = '2012.8.2.0'

extension = 'Skylight'
directory extension
CLEAN << extension

extension_files = FileList['{png,jpg,ttf,html,mp3,js}/*']

crx = "#{extension}.crx"
CLOBBER << crx
task :default => crx

directory 'js'
version = 'js/version.js'
extension_files << version
CLEAN << version

file version => [__FILE__, 'js'] do
  File.open(version, 'w') do |f|
    f.write "// Generated by Rake\nVERSION = '#{VERSION}';\n"
  end
end

# Compile coffee files
FileList['coffee/*.coffee'].each do |src|
  dest = src.pathmap '%{coffee,js}X.js'
  file dest => ['js', src] do
    sh "coffee --compile --bare --output js #{src}"
  end
  extension_files << dest
  CLEAN << dest
end

directory 'css'

# Compile less files
FileList['less/*.less'].each do |src|
  dest = src.pathmap '%{less,css}X.css'
  file dest => ['css', src] do
    sh "lessc #{src} #{dest} --yui-compress"
  end
  extension_files << dest
  CLEAN << dest
end

# Compile handlebars templates
templates = 'js/templates.js'
extension_files << templates
CLEAN << templates

file templates => FileList['js', 'handlebars/**/*.handlebars'] do
  sh "handlebars handlebars --output #{templates} --min"
end

directory 'json'
manifest = 'json/manifest.json'
extension_files << manifest
CLEAN << manifest

file manifest => [__FILE__, 'json'] do
  require 'json'

  hash = {
    manifest_version: 2,
    name: "Skylight",
    description: "A less flashy Skyrates.",
    version: VERSION,
    update_url: "http://skyrates.jusque.net/skylight/chromeupdate.xml",
    minimum_chrome_version: "21",
    icons: {
      "48"  => "Icon-48.png",
      "128" => "Icon-128.png"
    },
    permissions: ["notifications"],
    background: {
      scripts: ["background.js"]
    },
    content_scripts: [{
      matches:      ["http://skyrates.net/play.php"],
      css:          ["style.css"],
      js:           ["jquery.js", "underscore.js", "backbone.js",
                     "mousetrap.js", "handlebars.runtime.js", "templates.js",
                     "version.js", "constants.js", "prefs.js", "socket.js",
                     "alerter.js", "models.js", "helpers.js", "views.js",
                     "setup.js"],
      run_at:       "document_start",
      homepage_url: "http://skyrates.net/forum/viewforum.php?f=46"
    }],
    web_accessible_resources:
      FileList['{mp3,ttf,png,jpg}/*'].pathmap('%f').to_a
  }

  File.open(manifest, 'w') { |f| f.write JSON.pretty_generate hash }
end

extension_files.uniq.each do |src|
  dest = src.pathmap "#{extension}%s%f"
  file dest => [src, extension] do
    copy src, dest
  end
  file crx => dest
end

desc "Compile and marshall extension files."
file crx do
  puts <<-END

***
*** Build #{crx} from #{extension} directory in Chrome's Extensions manager
***
  END
  exit
end

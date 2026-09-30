require "bundler"
Bundler::GemHelper.install_tasks
require "rake/testtask"

task :default => :test

namespace :mirror do
  desc "Run the Gem::Mirror::Command"
  task :update do
    $:.unshift 'lib'
    require 'rubygems/mirror/command'

    mirror = Gem::Commands::MirrorCommand.new
    mirror.execute
  end

  task :latest do
    ENV["RUBYGEMS_MIRROR_ONLY_LATEST"] = "TRUE"
    Rake::Task["mirror:update"].invoke
  end

  desc "Unpack mirrored .gem files from DIR/gems into DIR/unpacked"
  task :unpack, [:dir] do |_, args|
    require 'rubygems/package'

    abort "usage: rake 'mirror:unpack[DIR]'" unless args[:dir]

    dir = File.expand_path(args[:dir])
    gem_paths = Dir.glob(File.join(dir, 'gems', '**', '*.gem')).sort

    gem_paths.each_with_index do |gem_path, index|
      target = File.join(dir, 'unpacked', File.basename(gem_path, '.gem'))
      next if File.directory?(target)

      partial = "#{target}.partial"
      FileUtils.rm_rf(partial)

      begin
        Gem::Package.new(gem_path).extract_files(partial)
        File.rename(partial, target)
        puts "[#{index + 1}/#{gem_paths.size}] #{File.basename(target)}"
      rescue StandardError => e
        FileUtils.rm_rf(partial)
        warn "Failed to unpack #{gem_path}: #{e.message}"
      end
    end
  end

  desc "Delete logs, binaries, JS, data, docs, images and audio from DIR/unpacked"
  task :prune, [:dir] do |_, args|
    abort "usage: rake 'mirror:prune[DIR]'" unless args[:dir]

    extensions = %w[
      .log .bin .jar .js .json .html .htm .csv .md
      .png .jpg .jpeg .gif .bmp .ico .svg .webp .tif .tiff .avif
      .mp3 .wav .ogg .flac .aac .m4a .wma .opus .aiff
    ]

    unpacked = File.join(File.expand_path(args[:dir]), 'unpacked')
    abort "Directory not found: #{unpacked}" unless File.directory?(unpacked)

    files = Dir.glob(File.join(unpacked, '**', '*'), File::FNM_DOTMATCH).select do |path|
      File.file?(path) && extensions.include?(File.extname(path).downcase)
    end

    bytes = files.sum { |path| File.size(path) }
    files.each { |path| File.delete(path) }

    puts "Deleted #{files.size} files (#{(bytes / 1024.0 / 1024).round(1)} MB) from #{unpacked}"
  end
end

Rake::TestTask.new

namespace :test do
  task :integration do
    sh Gem.ruby, '-Ilib', '-S', 'gem', 'mirror'
  end
end

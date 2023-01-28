require 'open-uri'

namespace :book do
  def exec_or_raise(command)
    puts `#{command}`
    if (! $?.success?)
      raise "[ERROR] '#{command}' failed"
    end
  end

  def generate_contributors_list(column_size)
    # Generating preformatted contributors list...
    `git shortlog -s | grep -v -E "(Straub|Chacon|dependabot)" | cut -f 2- | column -c #{column_size} > book/contributors.txt`
  end

  def download_locale(locale_file)
    locale_file_url = "https://raw.githubusercontent.com/asciidoctor/asciidoctor/master/data/locale/#{locale_file}"
    if not File.exist?(locale_file)
      puts "Downloading locale attributes file..."
      l10n_text = URI.open(locale_file_url).read
      File.open(locale_file, 'w') { |file| file.puts l10n_text }
      puts " -- Saved at #{locale_file}"
    else
      puts "Use existing file with locale attributes #{locale_file}"
    end
  end

  # Variables referenced for build
  lang = 'ru'
  locale_file = "attributes-#{lang}.adoc"
  date_string = Time.now.strftime('%d.%m.%Y')

  version_string = `git describe --tags`.chomp
  if version_string.empty?
    version_string = '0'
  end
  params = "--attribute revnumber='#{version_string}' --attribute revdate='#{date_string}' --attribute lang=#{lang} "
  ignore_urls = "'https://developer.github.com','https://developer.github.com/webhooks/','https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent','https://mvnrepository.com/artifact/org.eclipse.jgit/org.eclipse.jgit','https://www.mercurial-scm.org/'"

  # Tasks list
  desc 'build basic book formats'
  task :build => [:build_html, :build_epub, :build_fb2, :build_mobi, :build_pdf] do
    begin
      puts 'Validating generated files...'
      Rake::Task['book:check'].invoke
    end
  end

  desc 'prepare necessary data to start build'
  task :prebuild, [:column_size] do |t, args|
    args.with_defaults(:column_size => 96)

    download_locale(locale_file)
    generate_contributors_list(args.column_size)
  end

  desc 'build HTML format'
  task :build_html do
    Rake::Task['book:prebuild'].invoke(96)

    puts 'Converting to HTML...'
    `bundle exec asciidoctor #{params} -a data-uri progit.asc`
    puts ' -- HTML output at progit.html'
  end

  desc 'build EPUB format'
  task :build_epub do
    Rake::Task['book:prebuild'].invoke(48)

    puts 'Converting to EPUB...'
    `bundle exec asciidoctor-epub3 #{params} progit.asc`
    puts ' -- EPUB output at progit.epub'
  end

  desc 'build FB2 format'
  task :build_fb2 do
    Rake::Task['book:prebuild'].invoke(96)

    puts 'Converting to FB2...'
    `bundle exec asciidoctor-fb2 #{params} progit.asc`
    puts ' -- FB2 output at progit.fb2.zip'
  end

  desc 'build Mobi format'
  task :build_mobi do
    Rake::Task['book:prebuild'].invoke(96)

    puts 'Converting to Mobi (kf8)...'
    `bundle exec asciidoctor-epub3 #{params} -a ebook-format=kf8 progit.asc`
    puts ' -- Mobi output at progit.mobi'
  end

  desc 'build PDF format'
  task :build_pdf do
    Rake::Task['book:prebuild'].invoke(88)

    puts 'Converting to PDF... (this one takes a while)'
    `bundle exec asciidoctor-pdf #{params} progit.asc 2>/dev/null`
    puts ' -- PDF output at progit.pdf'
  end

  desc 'check HTML book'
  task :check_html do
    if not File.exist?('progit.html')
      Rake::Task['book:build_html'].invoke
    end

    puts ' -- Validate HTML file progit.html'
    exec_or_raise("bundle exec htmlproofer --url-ignore #{ignore_urls} --check-html progit.html")
  end

  desc 'check EPUB book'
  task :check_epub do
    if not File.exist?('progit.epub')
      Rake::Task['book:build_epub'].invoke
    end

    puts ' -- Validate EPUB output file progit.epub'
    exec_or_raise('bundle exec epubcheck progit.epub')
  end

  desc 'check generated books'
  task :check => [:check_html, :check_epub]

  desc 'clean all generated files'
  task :clean do
    begin
      puts 'Removing downloaded and generated files'

      FileList[locale_file, 'book/contributors.txt', 'progit.html', 'progit.epub', 'progit.fb2.zip' 'progit.pdf', 'progit.mobi'].each do |file|
        rm file
        rescue Errno::ENOENT
      end
    end
  end
end

task :default => "book:build"

namespace :book do
  def exec_or_raise(command)
    puts `#{command}`
    if (! $?.success?)
      raise "'#{command}' failed"
    end
  end

  desc 'build basic book formats'
  task :build do

    begin
      lang = "ru"
      begin
        l10n_text = open("https://raw.githubusercontent.com/asciidoctor/asciidoctor/master/data/locale/attributes-#{lang}.adoc").read
        File.open('attributes.asc', 'w') { |file| file.puts l10n_text}
        progit_txt = File.open('progit.asc').read
        if not progit_txt.include?("attributes.asc")
          progit_txt.gsub!('include::book/license.asc', "include::attributes.asc[]\ninclude::book/license.asc")
          File.open('progit.asc', 'w') {|file| file.puts progit_txt }
        end
      rescue
      end
      version_string = ENV['TRAVIS_TAG'] || `git describe --tags`.chomp
      if version_string.empty?
        version_string = '0'
      end
      date_string = Time.now.strftime("%d-%m-%Y")
      params = "--attribute revnumber='#{version_string}' --attribute revdate='#{date_string}' --attribute lang=#{lang} "
      
      puts "Generating contributors list"
      `git shortlog -s | grep -v -E "(Straub|Chacon|dependabot)" | cut -f 2- | column -c 96 > book/contributors.txt`

      puts "Converting to HTML..."
      `bundle exec asciidoctor #{params} -a data-uri progit.asc`
      puts " -- HTML output at progit.html"

      exec_or_raise('htmlproofer --check-html progit.html')

      puts "Converting to EPub..."
      `bundle exec asciidoctor-epub3 #{params} progit.asc`
      puts " -- Epub output at progit.epub"

      exec_or_raise('epubcheck progit.epub')
      
      # Commented out the .mobi file creation because the kindlegen dependency is not available.
      # For more information on this see: #1496.
      # This is a (hopefully) temporary fix until upstream asciidoctor-epub3 is fixed and we can offer .mobi files again.

      # puts "Converting to Mobi (kf8)..."
      # `bundle exec asciidoctor-epub3 #{params} -a ebook-format=kf8 progit.asc`
      # puts " -- Mobi output at progit.mobi"

      puts "Converting to PDF... (this one takes a while)"
      `bundle exec asciidoctor-pdf #{params} progit.asc 2>/dev/null`
      puts " -- PDF output at progit.pdf"

    end
  end
end

task :default => "book:build"

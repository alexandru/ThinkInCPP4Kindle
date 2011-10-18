# encoding: utf-8

require 'rubygems'
require 'bundler'
begin
  Bundler.setup(:default, :development)
rescue Bundler::BundlerError => e
  $stderr.puts e.message
  $stderr.puts "Run `bundle install` to install missing gems"
  exit e.status_code
end

require 'ruby-debug'
require 'nokogiri'
require 'pygments'
require 'cgi'
require 'erb'


task :clean do
  sh 'rm -rf build'
end

task :html_files_unzip => :clean do 
  mkdir_p 'build'
  FileList['assets/*'].each do |fpath|
    next if fpath =~ /opf/
    cp fpath, 'build'
  end
  sh 'unzip build/TICPP-2nd-ed-Vol-one-html.zip -d build/ 1>/dev/null'
  puts " --> unzipped archive"
end

task :build_html => :html_files_unzip do

  html_template = ''
  File.open('assets/html-template.html'){|fh| html_template = fh.read}
  html_template = ERB.new html_template

  to_process = FileList['build/*.html'].select{|x| x =~ /Chapter|Appendix/}.sort{|a,b| a =~ /Appendix/ && b =~ /Chapter/ ? 1 : (a =~ /Chapter/ && b =~ /Appendix/ ? -1 : a <=> b)}
  to_process.insert 0, 'build/Preface.html'

  to_process.each do |fpath|
    File.open(fpath) do |src|
      doc = Nokogiri::HTML(src)
      doc.css('br').remove
      doc.css('[face]').each{|x| x.remove_attribute 'face'}
      doc.css('[align]').each{|x| x.remove_attribute 'align'}

      if doc.css('div').first.to_html =~ /Table of Contents/
        doc.css('div').first.remove
      end

      if doc.css('div').last.to_html =~ /Last Update/
        doc.css('div').last.remove
      end

      html = doc.to_html
      html = html.gsub /<\/?(div|span|font)[^>]*>/, ''
      doc = Nokogiri::HTML(html)

      while true
        blanks = doc.css('*').select{|x| x.inner_html.strip == ''}
        break unless blanks.length > 0
        blanks.each do |elem|
          elem.remove
        end
      end

      unless fpath =~ /Preface/
        doc.css('blockquote pre').each do |elem|
          html = CGI.unescapeHTML(elem.inner_html)
          html = html.gsub /<\/?[^>]+>/, ''
          elem.inner_html = Pygments.highlight(html, :options => {:noclasses => true}, :lexer => 'cpp')
        end

        doc.css('blockquote pre div.highlight').each do |elem|
          elem.parent.parent.inner_html = elem.to_html
        end
      end

      body_html = doc.css('body').first.inner_html
      title = doc.css('title').first.inner_html

      if doc.css('h1').length == 0
        body_html = doc.css('body').first.inner_html
        body_html = "<h1>#{title}</h1>" + body_html
        doc.css('body').first.inner_html = body_html
      end

      if doc.css('h1').length > 1
        doc.css('h1')[1,].each{|x| x.remove}
        doc.css('h1:not(:first-child)').remove
      end

      number = 0
      doc.css('h1, h2').each do |elem|
        number += 1
        txt = elem.inner_html
        name = 'ref' + number.to_s
        elem.inner_html = "<a name='#{name}'>#{txt}</a>"
      end

      body_html = doc.css('body').first.inner_html
      final_html = html_template.result(binding)

      File.open(fpath, 'w') do |dest|
        dest.write(final_html)
      end

      puts "  --> processed #{fpath}"
    end
  end

  FileList['build/*.html'].select{|x| not to_process.include?(x)}.each do |fpath|
    rm fpath
  end
  FileList['build/*.txt'].each do |fpath|
    rm fpath
  end
end

task :generate_toc => :build_html do
  html_files = FileList['build/*.html'].select{|x| x =~ /Chapter|Appendix/}.sort{|a,b| a =~ /Appendix/ && b =~ /Chapter/ ? 1 : (a =~ /Chapter/ && b =~ /Appendix/ ? -1 : a <=> b)}
  html_files.insert 0, 'build/Preface.html'

  contents_html = "<ul id='toc'>\n"

  html_files.each do |fpath|
    html = File.read(fpath)
    doc = Nokogiri::HTML(html)
    stack = []
    
    fname = File.basename(fpath)
    h1_elem = doc.css('h1')
    h1_name = h1_elem.css('a').attr('name').value
    h1_text = h1_elem.css('a').inner_html.strip

    contents_html += "  <li><a href='#{fname}##{h1_name}'>#{h1_text}</a></li>\n"
    started = false

    doc.css('h2').each do |h2_elem|
      h2_name = h2_elem.css('a').attr('name').value
      h2_text = h2_elem.css('a').inner_html.strip.gsub(/\r?\n/, ' ').gsub(/\s+/, ' ')
      if not started
        contents_html += "  <li>\n"
        contents_html += "    <ul>\n"
        started = true
      end
      contents_html += "      <li><a href='#{fname}##{h2_name}'>#{h2_text}</a></li>\n"
    end

    if started
      contents_html += "    </ul>\n"
      contents_html += "  </li>\n"
    end
  end

  contents_html += "</ul>\n"
  
  File.open('build/toc.html', 'w') do |fh|
    html_template = ''
    File.open('assets/html-template.html'){|fh| html_template = fh.read}
    html_template = ERB.new html_template        

    title = "Table of Contents"
    body_html = "<h1>Table of Contents</h1>\n\n" + contents_html
    fh.write html_template.result(binding)
  end
  puts " --> generated table of contents"
end

task :build_opf => :generate_toc do
  template = ''
  File.open('assets/book.opf'){|fh| template = fh.read}
  template = ERB.new template
  
  html_items = FileList['build/*.html'].select{|x| x =~ /Chapter|Appendix/}.sort{|a,b| a =~ /Appendix/ && b =~ /Chapter/ ? 1 : (a =~ /Chapter/ && b =~ /Appendix/ ? -1 : a <=> b)}.map{|x| File.basename(x)}
  html_items.insert 0, 'Preface.html'

  graphic_items = FileList['build/*.gif'].map{|x| File.basename(x)}

  result = template.result(binding)
  File.open('build/book.opf', 'w') do |fh|
    fh.write result
  end
end

task :default => :build_opf do
  Dir.chdir('build') do
    puts `/opt/kindlegen/kindlegen book.opf -o thinking-in-cpp.mobi`
    mv "thinking-in-cpp.mobi", ".."
  end    
end

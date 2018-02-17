namespace :book do
  desc 'build basic book formats'
  task :build do

    puts "Generating contributors list"
    `git shortlog -s --all| grep -v -E "(Straub|Chacon)" | cut -f 2- | column -c 120 > book/contributors.txt`

    puts "Converting to HTML..."
    `bundle exec asciidoctor progit.asc`
    puts " -- HTML output at progit.html"

    puts "Converting to EPub..."
    `bundle exec asciidoctor-epub3 progit.asc`
    puts " -- Epub output at progit.epub"

    # sh "epubcheck progit.epub"

    puts "Converting to Mobi (kf8)..."
    `bundle exec asciidoctor-epub3 -a ebook-format=kf8 progit.asc`
    puts " -- Mobi output at progit.mobi"

    puts "Converting to PDF... (this one takes a while)"
    `bundle exec asciidoctor-pdf progit.asc 2>/dev/null`
    puts " -- PDF output at progit.pdf"
  end

  desc 'convert book to asciidoctor compatibility'
  task:convert do
    #`cp -aR ../progit2/images .`
    #`sed -i -e 's!/images/!!' .gitignore`
    #`git add images`
    #`git rm -r book/*/images`

    chapters = [
      ["01", "introduction"              ],
      ["02", "git-basics"                ],
      ["03", "git-branching"             ],
      ["04", "git-server"                ],
      ["05", "distributed-git"           ],
      ["06", "github"                    ],
      ["07", "git-tools"                 ],
      ["08", "customizing-git"           ],
      ["09", "git-and-other-scms"        ],
      ["10", "git-internals"             ],
      ["A",  "git-in-other-environments" ],
      ["B",  "embedding-git"             ],
      ["C",  "git-commands"              ]
    ]
    chaps =[
          ["C",  "git-commands"              ]
    ]

    crossrefs = {}
    chapters.each { | num, title |
      if num =~ /[ABC]/
        chap = "#{num}-#{title}"
      else
        chap = "ch#{num}-#{title}"
      end
      Dir[File.join ["book","#{num}-#{title}" , "sections","*.asc"]].map { |filename|
        File.read(filename).scan(/\[\[(.*?)\]\]/)
      }.flatten.each { |ref|
        crossrefs[ref] = "#{chap}"
      }
    }          
    
    headrefs = {}
    chapters.each { | num, title |
      if num =~ /[ABC]/
        chap = "#{num}-#{title}"
      else
        chap = "ch#{num}-#{title}"
      end
      Dir[File.join ["book","#{num}-#{title}", "*.asc"]].map { |filename|
        File.read(filename).scan(/\[\[(.*?)\]\]/)
      }.flatten.each { |ref|
        headrefs[ref] = "#{chap}"
      }
    }          
    
    # transform all internal cross refs
    chapters.each { | num, title |
      if num =~ /[ABC]/
        chap = "#{num}-#{title}"
      else
        chap = "ch#{num}-#{title}"
      end
      files = Dir[File.join ["book","#{num}-#{title}" , "sections","*.asc"]] +
              Dir[File.join ["book","#{num}-#{title}" ,"1-*.asc"]]
      p files
      files.each { |filename|
        content = File.read(filename)
        new_contents = content.gsub(/\[\[(.*?)\]\]/, '[[r\1]]').gsub(/<<(.*?)>>/) { |match|
          ch = crossrefs[$1]
          h = headrefs[$1]
          # p " #{match} -> #{ch}, #{h}"
          if ch
            # if local do not add the file
            if ch==chap
              "<<r#{$1}>>"
            else
              "<<#{ch}#r#{$1}>>"
            end
          elsif h
            if h==chap
              "<<#{chap}>>"
            else          
              "<<#{h}##{h}>>"
            end
          end
        }
        File.open(filename, "w") {|file| file.puts new_contents }
      }
    }     

    chapters.each { | num, title |
      if num =~ /[ABC]/
        chap = "#{num}-#{title}"
      else
        chap = "ch#{num}-#{title}"
      end
      Dir[File.join ["book","#{num}-#{title}" ,"1*.asc"]].map { |filename|
        content = File.read (filename)
        new_contents = content.gsub(/include::(.*?)asc/) {|match|
          "include::book/#{num}-#{title}/#{$1}asc"}
        File.open("#{chap}.asc", "w") {|file|
          file.puts "[##{chap}]\n"
          file.puts new_contents }
      }
    }
  end
end



task :default => "book:build"

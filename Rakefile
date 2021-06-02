# This file lets you pre-build any tikz figures appearing the the papers
# as PDF files, either for its own sake or to speed up compilation using
# the \tikzitdraft option in tikzit.sty.

# the name of the main file, with no extension
main_file = "template-quantum"

# the names of any files to scan for figures, with extension, one per line
file_names = %W(
  template-quantum.tex
)

# the name of your tikzstyles/tikzdefs file, no extension
tikzstyles = "quantum"


######################################################################
require "listen"

# compile a list of figures based on file_names
figures = []
file_names.each do |fname|
  f = open(fname)
  open(f).each_line do |line|
    line.scan(/tikzfig\{([^\}]*)\}/).each do |tikz|
      figures << tikz[0]
    end
  end
  f.close
end

# get a list of all the tikz figures in the figures/ dir
tikzfiles = FileList["figures/*.tikz"].pathmap("%n")
exists = {}

# filter out figures that don't actually exist (e.g. placeholders in tex files)
tikzfiles.each { |f| exists[f] = true }
outfiles = figures.map do |f|
  "cache/#{f}.pdf" if exists.include? f
end.compact

task :default => [:prep, :pfigs]

task :clean do
  rm_r "cache"
end

task :listfigs do
  puts outfiles
end

task :listen do
  listener = Listen.to("./figures") do
    puts "Rebuilding figures..."
    t = Rake::Task["all"]
    t.invoke
    t.reenable
    t.all_prerequisite_tasks.each { |t1| t1.reenable }
    puts "done."
  end
  puts "Listening for changes in ./figures"
  listener.start
  sleep
end

task :prep => ["cache", "cache/tikzit.sty"]
directory "cache"
file "cache/tikzit.sty" => "tikzit.sty" do
  cp "tikzit.sty", "cache/tikzit.sty"
end

multitask :pfigs => outfiles

tikzfiles.each do |fig|
  file "cache/#{fig}.pdf" => "figures/#{fig}.tikz" do
    tex = <<~TEX
    \\documentclass{article}
    \\usepackage{xr}
    \\externaldocument{../#{main_file}}
    \\externaldocument{../Output/#{main_file}}
    \\usepackage{tikzit}
    \\input{../#{tikzstyles}.tikzstyles}
    \\input{../#{tikzstyles}.tikzdefs}
    \\usepackage[graphics,active,tightpage]{preview}
    \\PreviewEnvironment{tikzpicture}

    \\begin{document}
    \\input{../figures/#{fig}.tikz}
    \\end{document}
    TEX

    File.open("cache/#{fig}.tex", "w") { |f| f.write(tex) }
    system("pdflatex --interaction=nonstopmode \"#{fig}.tex\"", :chdir=>"cache")
  end
end

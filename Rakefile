require 'listen'

mainfile = 'main'
fnames = %W(
  abstract.tex
  optimisation.tex
  stab2.tex
  circuit.tex
  phase-poly.tex
  stabiliser.tex
  correctness.tex
  main.tex
  preamble.tex
  synth.tex
  intro.tex
  mbqc.tex
  simulation.tex
  zx.tex
)
tikzstyles = "circuits"

figures = []
fnames.each do |fname|
  f = open(fname)
  open(f).each_line do |line|
    line.scan(/tikzfig\{([^\}]*)\}/).each do |tikz|
      figures << tikz[0]
    end
  end
  f.close
end

tikzfiles = FileList['figures/*.tikz'].pathmap('%n')
exists = {}
tikzfiles.each { |f| exists[f] = true }
outfiles = figures.map do |f|
  "cache/#{f}.pdf" if exists.include? f
end.compact

######################################################################

task :figs => [:prep, :pfigs]

task :clean do
  rm_r "cache"
end

task :listen do
  listener = Listen.to('./figures') do
    puts "Rebuilding figures..."
    t = Rake::Task['all']
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
  sh "cp tikzit.sty cache/"
end


file "#{mainfile}.pdf" => "#{mainfile}.tex" do
  sh "latexmk -synctex=1 -pdf #{mainfile}.tex"
end

multitask :pfigs => outfiles

tikzfiles.each do |fig|
  file "cache/#{fig}.pdf" => "figures/#{fig}.tikz" do
    tex = <<~TEX
    \\documentclass{article}
    \\usepackage{xr}
    \\externaldocument{../#{mainfile}}
    \\externaldocument{../Output/#{mainfile}}
    \\usepackage{tikzit}
    \\input{../#{tikzstyles}.tikzstyles}
    \\input{../#{tikzstyles}.tikzdefs}
    \\usepackage[graphics,active,tightpage]{preview}
    \\PreviewEnvironment{tikzpicture}

    \\begin{document}
    \\input{../figures/#{fig}.tikz}
    \\end{document}
    TEX

    File.open("cache/#{fig}.tex", 'w') { |f| f.write(tex) }
    sh "(cd cache; pdflatex --interaction=nonstopmode #{fig}.tex)"
  end
end

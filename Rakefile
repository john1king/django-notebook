
require "erb"

def title(filename)
  open(filename, 'r') do |f|
    f.each_line do |line|
      return $1 if line =~ /^#\s*(.*)/
    end
  end
  File.basename(filename, '.*')
end

Context = Struct.new(:notes) do

  def render(template)
     ERB.new(File.read(template), nil, '-').result binding
  end

end
Note = Struct.new(:title, :filename, :date)

task :index do
  Dir.chdir(File.dirname(__FILE__)) do
    context = Context.new([])
    Dir.glob('notes/**/*.md').each do |note|
      date = `git log -1 --format=%ai notes/signal.md`
      context.notes.push Note.new(title(note), note, date[0..9])
    end
    File.write('README.md', context.render('README.md.erb'), mode: 'w')
  end
end

task :default => :index

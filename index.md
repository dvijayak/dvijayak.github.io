---
title: Under Construction
---

~~~ ruby
reader = Reader.new

reader.wait do |done|
  unless done
    puts "Got to wait...nevertheless, feel free to peruse the site so far..."
  end
end
~~~
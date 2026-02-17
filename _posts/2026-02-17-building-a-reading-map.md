---
title: Building a Reading Map
date: 2026-02-17
published: true
---

A few months ago I went back to Goodreads after I picked up a popular book to see what people thought of it. I'd first signed up for Goodreads in 2010 to keep track of my reading and stopped using it altogether around 2021. In the meantime it boomed and turned into a momentum machine for the publishing industry. 

I like the idea of tracking my reading and having a database to come back to, and I slightly regret that I stopped doing it. But was Goodreads it? I probably had a good reason for abandoning it.  I'm generally not interested in the kind of reviews that Goodreads hosts and encourages - ARC holders who write the same lukewarm platitudes, snarky gotchas - even though I appreciate this must be invaluable for publishers. There are several other platforms and apps that let you do a bunch of other things that most readers value, such as tracking your reading time, or getting personalized recommendations for your next book based on your current and past reads. I'm not interested in any of them either. I generally don't believe in self-optimization of that degree and I already know very well what I want to read next. So none of the currently available products satisfy me. Plus I'd like to add other forms of reading to my database, such as poetry, or even articles (note to myself: renew your LRB subscription).


So I built a tool that would capture the reading experience as I actually care for: not just a collection of metadata but the references I chase down, the quotes I love and would like to remember later, and ideally, the connections between ideas as I read, so that over time a mind map emerges across all of my reading. At the moment, the database structure looks something like this: 

![Reading map diagram](/assets/images/reading-map2.png)


And after a few days of data entry, this is my context db: 



![Valis context db](/assets/images/valis-screenshot2.png)

Quotations have no tags, but references are labeled with a few of them using Ollama. They are a bit rough around the edges at the moment (for example: some phrases with underscore, some not), and it's something that I will need to return to later. I'm also not sure what I'd ultimately do with tags, whether they'll be useful for what I ultimately want to build: a graph-like structure that surfaces connections across ideas, cross references, names and places across different works. One other hope I have from this data approach is to see how my engagement with a book differs across genres, languages, topics. I'm currently reading two novels simultaneously, and while I keep adding stuff from one, the other is just... reading with no urge to record anything from it. I'd like to see what this says about how I remember my reading experience in the long run. 

And oh yes: it's a cli tool for now and that's intentional. I come up with better ideas and can see a project's shortcomings better from the terminal. A GUI would probably take the attention away from the mechanics to how things look. Eventually I'll need to move out of the terminal and build an interface, but that's a problem for later.

I'm going to keep on reading and build out my book db, at the same time I'll continue experimenting with tag generation, text extraction from images and more context expansion using Ollama. Prompt engineering and working with models in this capacity are not something I have experience with. I'm excited to see where it takes me though.


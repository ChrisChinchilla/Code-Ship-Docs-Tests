# tbc

What's one of the first things you look at when trying a new piece of software? Or after you've hit that tempting 'download' button, what's your usual next step?

I will take a bet that for at least 70% of you it's the documentation. So why is documentation usually one of the last things that developers want to handle, or enjoy so little. Someone recently said to me:

> Documentation is like hiusework, we all know it needs to be done, but few actually want to do it.

Unless your someone like me, I think this quote is a pretty good summary of how most developers think about writing docs. this is not a post about writing better documentation, for that I recommend my popular post on medium, rather a guide for automating some of the boring tasks. This includes spelling and linting, all of which can be run locally or through continuous integration tools.

## Spelling

spelling and grammar help give your documentation polish, and even though we have had spell checkers for years, mistakes still slip through, and even as a native English speaker I still make a lot of mistakes. I am going to assume that your documentation is in plain text formats (markdown, restructured Text etc) and that you likely use some form of text editor, I personally use atom, but similar should apply for any other. The advantage of using plain text formats and more 'open' text editors is that you can often use the same tool chain on your desktop as on your ci servere, including sharing custom dictionaries in version control.

## grammar and writing better

English has grammar rukes, that unless you break intentionally, should be adhered to or clear writing. But there are also style guides and good practices for better writing, that you may or may not decide to follow.

This starts to get into the Territory of linting, which is a process of providing guidelines for improevment, but I'll return to that later.

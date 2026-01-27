Google dorks  


| ilter                                   | Description                                                                               | Example                                               |
| :-------------------------------------- | :---------------------------------------------------------------------------------------- | :---------------------------------------------------- |
| allintext                               | Searches for occurrences of all specified keywords.                                       | `allintext:"keyword"`                                 |
| intext                                  | Searches for the occurrence of keywords at once or consecutively.                         | `intext:"keyword"`                                    |
| intitle                                 | Searches for occurrences of keywords in the title all or one.                             | `intitle:"keyword"`                                   |
| allintitle                              | Searches for all occurrences of keywords at once.                                         | `allintitle:"keyword"`                                |
| inurl                                   | Searches for a URL that matches one of the keywords.                                      | `inurl:"keyword"`                                     |
| allinurl                                | Searches for a URL that matches all the keywords in the query.                            | `allinurl:"keyword"`                                  |
| site                                    | Searches specifically for that particular website and lists all results for that website. | `site:"www.github.com"`                               |
| filetype                                | Searches for a specific file type named in the query.                                     | `filetype:"pdf"`                                      |
| link                                    | Searches for external links to pages.                                                     | `link:"keyword"`                                      |
| numrange                                | Used to find specific numbers in your search.                                             | `numrange:33-43`                                      |
| before/after                            | Used to search within a specified date range.                                             | `filetype:pdf & (before:2021-01-01 after:2021-05-01)` |
| allinanchor (and also inanchor)         | This shows the websites that the keywords refer to in links, in order of most links.      | `inanchor:rat`                                        |
| allinpostauthor (and also inpostauthor) | Exclusively for the blog search, blog posts written by specific people are picked out.    | `allinpostauthor:"keyword"`                           |
| related                                 | List web pages that are "similar" to a given web page.                                    | `related:www.github.com`                              |
| cache                                   | Displays the version of the web page that Google has in its cache.                        | `cache:www.github.com`                                |
OR  
site:instagram.com | site:github.com  
AND  
site:github.com & site:twitter.com  
Combine  
(site:instagram.com | site:twitter.com) (intext:"admin")  
(site:instagram.com | site:twitter.com) & intext:"admin"  
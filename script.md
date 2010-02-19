INTRO
-----

Bankjob is a command-line ruby program for scraping online banking sites and
producing statements in OFX or CSV formats. This is useful if you want to get
your banking data into an accounting program, such as FreeAgent, and if your
bank doesn't already provide the necessary support for exporting your data.

I bank with HBOS. They let me get my banking data in csv or microsoft money
format, but only by exporting my monthly 'paper-saving' statement. Yes,
MONTHLY paper-saving statement. Online banking saves time and money, by
letting me view monthly paper-saving statements on the internet.

I don't want my statements monthly. The whole point of using accounting
software is to keep track of what my finances look like right now. So if I
want my banking data in real time, I have to log in to my bank's shoddy web
site. Luckily we can scrape this data with Bankjob and associates.

STRUCTURE
---------

Mechanize is a ruby library that provides a virtual browser. It automatically
stores and sends cookies, allows you to populate fields and submit forms
programatically. This makes it possible for BankJob to negotiate the login
steps for an online banking website.

HPricot is an HTML parser, which allows you to extract content from an HTML
document using XPath and CSS selectors, much like jQuery. This makes it
possible for BankJob to scrape data from account info screens once logged in.
Note that you could replace HPricot with the Nokogiri library if you prefer.

BankJob models the business logic with a handful of ruby classes. It allows
you to build a Statement, which consists of a collection of Transaction
objects. These classes implement `.to_csv` and `.to_ofx` methods, allowing for
easy output into those formats.

BankJob also includes a BaseScraper class, which provides a template for
writing your own scrapers. You must implement the `fetch_transactions_page`
method, which should log in to the online banking site, and the
`parse_transactions_page` method, which creates a Statement from the account
data on screen.

SCRAPING
--------

There are two problems to be solved in writing a scraper: getting past the
login screen, and grabbing the data to build a statement. But you don't have
to solve the problems in that order. If you manually log in to your online
banking site, you can just save the source code for your statement page to a
file, and pass this directly to bankjob for testing.

Once you've saved your sample statement as an HTML file, you can run bankjob
passing it this file with the `--input` option. When BankJob is run with this
option, it skips the `fetch_transactions_page` step, and just runs
`parse_transactions_page` directly on the file it has been passed.

Create Statement, find table, iterate through rows building transactions, add
to statement.

Each cell in the table row is tidied up. Most of these values can then be 
assigned directly to the transaction. In this statement, amount is represented 
by money in and money out columns. So I check which of these contains a 
number, and make it negative if it's the money out column.

LOGGING IN
----------

Now we come to the interesting part of the problem: logging in. My first
mistake was forgetting that this is for real! Each time I ran bankjob against
the login screen for my online banking, Mechanize was submitting the form with
invalid paramaters. I was locked out of my online banking, until I could prove
that I was me again.

So I copied the HTML source from the online banking form into a template,  and
built a simple Sinatra app to simulate the login process. POSTing to the
homepage shows a 'welcome' message if you provide the correct username,
password and one of the valid answers to the security question.

The HBOS login form poses a problem: the security question cycles between 5
different options. A possible solution would be to send the correct response
from a set of saved answers. This would allow for the script to be scheduled,
but it would be a disaster in terms of security. 

Seeing as bankjob is a command line tool, I decided that it would be acceptable
to prompt the user for the appropriate response. 

The highline gem makes it really easy to write interactive command line
prompts. Instead of using ruby's low level `puts` and `gets` methods, you can
use the `ask` command. This can perform validation on user input, and it can
mask the characters typed at the command line.

Here is the `login` method from my HBOS scraper. As you can see, it uses
highline to prompt the user for username and password. The final prompt is the
security question, whose text is fetched from the page using HPricot. Input to
the form fields is saved in the Mechanize agent, which also submits the form. 

USAGE
-----

Once you've written a scraper, you can run bankjob specifying the path to it
with the `--scraper` option. The `--csv` option can be used to specify the
path to a directory where the comma separated value file will be saved. It
automatically generates a filename based on the range of dates covered by the
statement.

Bankjob has built in support for uploading statements to Wesabe, which is a
personal finance web application. If you include the `--wesabe` option when
you run bankjob, providing username, password and your wesabe account,
then the OFX file will be automatically uploaded to wesabe.

Wesabe just happens to be supported out the box. BankJob could upload OFX data
to any web app with an API, such as FreeAgent. If you want to write a scraper
for your online banking site, I recommend using bankjob.  Any questions?
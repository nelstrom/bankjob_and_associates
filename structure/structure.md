!SLIDE bullets

# MECHANIZE #

* store & send cookies
* populate fields
* submit forms

!SLIDE bullets

# HPRICOT #

* HTML parser
* XPath & CSS selectors (jQuery)

!SLIDE small

# BANKJOB #

    @@@ ruby
    class Statement
      has_many :transactions
      
      def to_csv; end
      def to_ofx; end
    end
    
    class Transaction
      belongs_to :statement
      
      def to_csv; end
      def to_ofx; end
    end

!SLIDE code

    @@@ ruby
    class MyBankScraper < BaseScraper

      currency  "GBP"
      decimal   "."

      def fetch_transactions_page(agent)
        # login to online banking
      end

      def parse_transactions_page(agent)
        # scrape it!
      end

    end
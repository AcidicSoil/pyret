include shared-gdrive(
  "cs111-2020.arr",
  "1imMXJxpNWFCUaawtzIJzPhbDuaLHtuDX")
include gdrive-sheets
include data-source # to get the sanitizers
include tables
import math as M
import statistics as S
import lists as L

ssid = "1DKngiBfI2cGTVEazFEyXf7H4mhl8IU5yv2TfZWv6Rc8"
cleaned-data =
  load-table: name, email, tickcount, discount, delivery
    source: load-spreadsheet(ssid).sheet-by-name("Cleaned", true)
    sanitize name using string-sanitizer
    sanitize email using string-sanitizer
    sanitize tickcount using num-sanitizer
    sanitize discount using string-sanitizer
    sanitize delivery using string-sanitizer
  end

event-data =
  load-table: name, email, tickcount, discount, delivery
    source: load-spreadsheet(ssid).sheet-by-name("Data", true)
    sanitize name using string-sanitizer
    sanitize email using string-sanitizer
    sanitize tickcount using num-sanitizer
    sanitize discount using string-sanitizer
    sanitize delivery using string-sanitizer
  end


fun equal-pickup(r :: Row) -> Boolean:
  r["delivery"] == "pickup"
where:
  equal-pickup(cleaned-data.row-n(0)) is false
  equal-pickup(cleaned-data.row-n(6)) is true
end

#creates table 
pickup-table = filter-with(cleaned-data, equal-pickup)

#creates list from new table
pickup-list = pickup-table.get-column("name")


fun org-address(email :: String) -> String:
  doc: "determine whether email has .org"
  string-split(email, ".").get(1)
where:
  org-address("bonnie@pyret.org") is "org"
  org-address("parrot@web.com") is "com"
end 
  

#finds if email has org
fun web-com-address(email :: String) -> Boolean:
  doc: "determine whether email is from org"
   string-split(email, ".").get(1) == "org"
where:
  web-com-address("bonnie@pyret.org") is true
  web-com-address("parrot@web.com") is false
end


#takes avg tickcounts of emails ending in org 
email-tick-count-org =
    add-row(
    add-row(cleaned-data.empty(),
      cleaned-data.row-n(2)),
    cleaned-data.row-n(6))
#creates list 
tickcounts = email-tick-count-org.get-column("tickcount")
S.median(tickcounts)



#function that checks if tickcount is gt 8 and contains org
fun greater-than-8-tc(r :: Row) -> Boolean:
  if (r["tickcount"] > 8) and (string-contains(r["email"], "org")):
    true
    else:
    false
  end
where:
  greater-than-8-tc(event-data.row-n(0)) is false
  greater-than-8-tc(event-data.row-n(1)) is false
end

  
#creates table showing all names with a tickcount gt 8
gt-8-tc = filter-with(event-data, greater-than-8-tc)

print(gt-8-tc)
#finds number of rows in table with gt 8 tc and org email
gt-8-tc.length()
#converts above to list and does the same
emails = gt-8-tc.get-column("email")
L.length(L.filter(org-address, emails))

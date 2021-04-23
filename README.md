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

codes = cleaned-data.get-column("discount")

L.distinct(codes)

#select-column makes a table
select-columns(cleaned-data, [list: "tickcount"])

#get-column makes a list
tickcounts = cleaned-data.get-column("tickcount")

fun real-code(c :: String) -> Boolean:
  not(c == "none")
end
L.filter(real-code, codes)
#L.filter(lam(c): not(c == "none") end, codes)

fun low-case(c :: String) -> Boolean:
  not(c == string-to-lower("none"))
end
L.filter(low-case, codes)

fun web-com-address(email :: String) -> Boolean:
  doc: "determine whether email is from web.com"
  string-split(email, "@").get(1) == "web.com"
where:
  web-com-address("bonnie@pyret.org") is false
  web-com-address("parrot@web.com") is true
end

emails = cleaned-data.get-column("email")

L.length(L.filter(web-com-address, emails))

fun extract-username(email :: String) -> String:
  doc: "extract the portion of an email address before the @ sign"
  string-split(email, "@").get(0)
where:
  extract-username("bonnie@pyret.org") is "bonnie"
  extract-username("parrot@web.com") is "parrot"
end

L.map(extract-username,
  [list: "parrot@web.com", "bonnie@pyret.org"])

fun pickup(r :: Row) -> Boolean:
  doc: "determines if they will pickup ticket"
  will-pickup = r["delivery"]
  if will-pickup == "pickup": true
  else: false
  end
end

fun chk-dlv-stat(t :: Table) -> Table:
  doc: "function for creating a table to work with"
  transform-column(t, "delivery", pickup)
end

will-pickup = build-column(cleaned-data, "will-pickup", pickup)

pickup-list = cleaned-data.get-column("names")

fun yes-pickup(pickup-list :: List) -> List:
  map(lam(p): if (p == "pickup"):
        pickup-list = cleaned-data.get-column("names")
    else:
      p
    end
  end, pickup-list)
end
#L.filter(lam(p): (p == "pickup") end, pickup-list)



stir-fry =
  [list: "peppers", "pork", "onions", "rice"]
dosa = [list: "rice", "lentils", "potato"]
misir-wot =
  [list: "lentils", "berbere", "tomato"]

examples:
  recipes-uses(stir-fry, "pork") is true
  recipes-uses(stir-fry, "tomato") is false
end

fun recipes-uses(lst :: List<String>, ing :: String) -> Boolean:
  filter(lam(x):
      x == ing
    end, lst).length() > 0
end

fun make-veg(lst :: List<String>) -> List<String>:
  map(lam(x):
      if (x == "pork") or (x == "beef") or (x == "chicken"):
      "tofu"
    else:
      x
    end
  end, lst)
end

fun pro-veg-count(lst :: List<String>) -> Number:
  filter(lam(x):
      not((x == "rice") or (x == "noodles"))
    end, lst).length()
end

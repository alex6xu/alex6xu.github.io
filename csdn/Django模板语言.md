[java] view plain copy

Dear {{ person_name }},



Thanks for placing an order from {{ company }}. It's scheduled to   ship on {{
ship_date|date:"F j, Y" }}.



Here are the items you've ordered:



   {% for item in item_list %}

  * {{ item }}
   {% endfor %}      {% if ordered_warranty %}

Your warranty information will be included in the packaging.

   {% endif %}

Sincerely,{{ company }}




Show estimated shipping  on Cart drawer:- 
Check steps Below.
1. Need to create settings under cart settings under config dolder under settings.
 {
        "type": "text",
        "id": "shipping_price",
        "label": "Delivery Fee",
        "default": "350"
      },
      {
        "type": "checkbox",
        "id": "calculation_of_free_shipping",
        "label": "Calculate shipping fees in cart?",
        "info": "Make sure that you have properly configured your delivery fees in the field 'Delivery fee' at the bottom.",
        "default": false
      },
      {
        "type": "text",
        "id": "cart_free_shipping_threshold",
        "label": "Free shipping minimum amount",
        "info": "Enter amount using number only. If using multiple currencies, separate each currency code with its amount (eg.: USD:50,EUR:60,JPY:9000)"
      },

2. Find MiniCart-Drawer and add below variables to this file.
{% assign total_price    = cart.total_price %}
{%  assign shippingcharge = settings.shipping_price | money %}
{% comment %} CHECK FREE SHIPPING AMOUNT {% endcomment %}
{% if settings.cart_free_shipping_threshold != blank %}
    {% assign cart_free_shipping_threshold = settings.cart_free_shipping_threshold | times: 100 %}
{% else %}
    {% assign cart_free_shipping_threshold = false %}
{% endif %}
{% comment %} END {% endcomment %}

{% assign cart_total_price = total_price | divided_by: 100 %}
{% assign freeshipamt = settings.cart_free_shipping_threshold | plus: 0 %}

{% if settings.shipping_price != blank %}
    {% assign shipping_price = settings.shipping_price %}
{% else %}
    {% assign shipping_price = 0 %}
{% endif %}
{% comment %} END {% endcomment %}

{% comment %} CHECK IF CALCULATION IS ENABLED {% endcomment %}
{% if settings.calculation_of_free_shipping == false %}
    {% assign shipping_price = 0 %}
{% endif %}
{% comment %} END {% endcomment %}

{% comment %} IF FREE SHIPPING AMOUNT IS REACHED {% endcomment %}
{% if cart_total_price >= freeshipamt and settings.cart_free_shipping_threshold != blank %}
    {% assign shipping_price = 0 %}
{% endif %}

3. Find the Price case or code in cart drawer file and implement  the below code as per the theme structure. 

{% if settings.calculation_of_free_shipping != false %}
              {% if settings.shipping_price != blank %}
                  <div class="t4s-cart-total t4s-row t4s-gx-5 t4s-gy-0 t4s-align-items-center t4s-justify-content-between" {{ block.shopify_attributes }}>
                     <div class="t4s-col-auto"><strong>{{ 'cart.mini_cart.subtotal' | t }}</strong></div>
                     <div data-cart-prices class="t4s-col-auto t4s-text-right">
                        {%- if cart.total_discount > 0 -%}
                        <div class="t4s-cart__originalPrice t4s-d-inline-block">{{ cart.original_total_price | money }}</div>
                        <div class="t4s-cart__discountPrice t4s-d-inline-block"> - {{ cart.total_discount | money }}</div>
                        {%- elsif compare_tt_price > total_price and false -%}
                        <div class="t4s-cart__originalPrice t4s-d-inline-block">{{ compare_tt_price | money }}</div>
                        <div class="t4s-cart__discountPrice t4s-d-inline-block"> - {{ compare_tt_price | minus:total_price | money }}</div>
                        {%- endif -%}
                        <div id="cart-subtotalp" data-sub-total="{{ total_price }}" class="t4s-cart__totalPrice cart-subtotal">{{ total_price | money }}</div>
                     </div>
                     <div class="t4s-col-auto"><strong>{{ 'cart.mini_cart.shipment' | t }}</strong></div>
                      <span class="totals__subtotal-value totals__subtotal-value-price shiping-text-{% if shipping_price == 0 %}g{% else %}p{% endif %}" data-shiping-amount="{{ settings.shipping_price }}" data-shiping-price="{{ shippingcharge }}">
						{% if shipping_price == 0 %}
						Free
						{% else %}
						{{ shipping_price | money }}
						{% endif %}
						</span>
						
                        <div id="total-amount" class="t4s-col-auto"><strong>{{'cart.mini_cart.alltotal' | t}}</strong></div>
                        <div data-cart-prices class="t4s-col-auto t4s-text-right">
                        {%- if cart.total_discount > 0 -%}
                        <div class="t4s-cart__originalPrice t4s-d-inline-block">{{ cart.original_total_price | money }}</div>
                        <div class="t4s-cart__discountPrice t4s-d-inline-block"> - {{ cart.total_discount | money }}</div>
                        {%- elsif compare_tt_price > total_price and false -%}
                        <div class="t4s-cart__originalPrice t4s-d-inline-block">{{ compare_tt_price | money }}</div>
                        <div class="t4s-cart__discountPrice t4s-d-inline-block"> - {{ compare_tt_price | minus:total_price | money }}</div>
                        {%- endif -%}
                        {% comment %} CALCULATE ALWAYS TOTAL CART PRICE {% endcomment %}
                        {% assign final_cart_total_price = total_price | plus: shipping_price %}
                        {% comment %} END {% endcomment %}
                        <div class="t4s-cart__totalPrice carttotalamount">{{ final_cart_total_price | money }}</div>
                     </div>
                    {% endif %}
                {% endif %}
                  </div>

4. Open the js file and paste our js to custom js file or own file. You can also paste this code cart drawer bottom of the  file.

{% comment %} IF CALCULATION OF DELIVERY FEES IS DEACTIVATED {% endcomment %}
{% if settings.calculation_of_free_shipping != false %}
    {% if settings.shipping_price != blank %}
        <script>
           
$(document).ready(function() {

  $(document).on("click", ".t4s-mini_cart__actions .is--minus, .t4s-mini_cart__actions .is--plus", function() {
    updateCartSubtotal();
  });

              function updateCartSubtotal() {
              
              var shippingPrice = $("span.shiping-text-p").attr('data-shiping-price');
              var shiping_amount = parseFloat($("span.shiping-text-p").attr('data-shiping-amount')) / 100; // Convert to a number and divide by 100
              
              var freeshipprice = {{cart_free_shipping_threshold }};
             
              setTimeout(function() {
              
              var cartTotal =  $("div#cart-subtotalp").attr('data-sub-total');
              var shipingTextSpan = $('.shiping-text-p');
              
              if (cartTotal > freeshipprice) {
                $("span.shiping-text-p").addClass('shiping-text-g');
              shipingTextSpan.text("Free");
              console.log('Free');
              } else {
                $("span.shiping-text-p").removeClass('shiping-text-g');
             var cartTotala = parseFloat($("div#cart-subtotalp").attr('data-sub-total')) / 100; // Convert to a number and divide by 100
              var final_cart_total_price_amount = cartTotala + shiping_amount; // Calculate the sum of the two variables
              $('#total-amount').next('div').addClass('updated-cart-amount');
                $('.updated-cart-amount').find('.cart-subtotal').html("€"+final_cart_total_price_amount);
              $('.shiping-text-p').text(shippingPrice);
            
              }
              
              }, 3000);
              }
});
 </script>
    {% endif %}
{% endif %}
{% comment %} END {% endcomment %}

5. Open Css File and paste below css to own css and create custom.css for this 
<style>
	  .shiping-text-g{
  color:#79AC2B;
}

.t4s-drawer__bottom  .t4s-cart-total {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 10px;
}
.t4s-drawer__bottom .t4s-col-auto,
.t4s-drawer__bottom .totals__subtotal-value {
    font-size: 15px !important;
  line-height: normal;
}
.t4s-drawer__bottom .totals__subtotal-value {
  text-align: right;
}
.t4s-drawer__bottom .t4s-col-auto strong,
.t4s-drawer__bottom .totals__subtotal-value,
.t4s-drawer__bottom .t4s-cart__totalPrice {
    font-weight: 400 !important;
}
</style>

6. If your theme name is Kalles, then it's work fine, if your theme is not Kalles then you need to modify the code as per theme structure.

View Url:- https://www.true-nature.de/
Preview url:- https://www.true-nature.de/?_ab=0&_fd=0&_sc=1&preview_theme_id=146305253641


if you face any issue contact with ecommerce.web.expert@gmail.com

Thanks for View.

{% assign curPostCategory = '' %}
{% assign prePostcategory = '' %}

<aside class="mod rightCol">

    <ul class="menu mT">
    {% for cat in (page.sidebar_categories) %}
    
        <li id="{{ cat }}">
        
            <a href="#{{ cat }}">{{ cat | replace:'_',' ' | capitalize }}</a>
            
            <ul class="js-guides">
        {% for post in site.posts reversed %}
        
            {% if post.categories contains 'photon-cloud' %}
            {% if post.categories contains cat %}
            
                {% capture curPostCategory %}{{ post.title | replace:' ','_' }}{% endcapture%}
                
                {% if post.subtitle %}
            
                    {% if prePostCategory == curPostCategory %}
                        {% capture addCssClass %}2ndlevel{% endcapture %}
                    {% else %}
                        {% assign prePostCategory = curPostCategory %}
                        {% assign addCssClass = '' %}
                    {% endif %}
            
                {% endif %}
                
                {% if page.title == post.title %}
                    {% capture addCssClass %}{{ addCssClass }} active{% endcapture %}
                {% endif %}
                
                
                    <li class="{{ addCssClass }}" data-postcategory="{{ curPostCategory }}">
                        <a href="{{ post.url }}{{ GET }}#cat-{{ cat }}">{{ post.title }}</a>
                    </li>
            
            {% endif %}
            {% endif %}
            
        {% endfor %}    
            </ul>
        </li>
            
    {% endfor %}
    </ul>

</aside>
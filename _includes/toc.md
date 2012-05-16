<aside>
    <nav>
    {% for product in (page.main_categories) %}
        {% if product == 'photon-server' %}
            {% assign GET = '?p=1' %}
        {% else %}
            {% assign GET = '?p=2' %}
        {% endif %}
    
        <ul class="hide" id="nav-{{ product }}">
        {% for cat in (page.sidebar_categories) %}
        
                <li>
                    <h3>
                        <a href="#">{{ cat | replace:'_',' ' | capitalize }}</a>
                    </h3>
                    
                    <ul class="js-guides">
                {% for post in site.posts reversed %}
                
                    {% if post.categories contains page.main_category %}
                    {% if post.categories contains cat %}
                        {% if page.title == post.title %}
                            <li class="disable"><span>{{ page.title }}</span></li>
                        {% else %}
                            <li><a href="{{ post.url }}{{ GET }}">{{ post.title }}</a></li>
                        {% endif %}
                    {% endif %}
                    {% endif %}
                    
                {% endfor %}
                    </ul>
                </li>
                
        {% endfor %}
        </ul>
    
    {% endfor %}
    </nav>
</aside>
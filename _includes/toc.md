<aside class="mod rightCol">

    <ul class="menu mT hide" id="nav-{{ product }}">
    {% for cat in (page.sidebar_categories) %}
    
            <li id="{{ cat }}">
            
                <a href="#{{ cat }}">{{ cat | replace:'_',' ' | capitalize }}</a>
                
                <ul class="js-guides">
            {% for post in site.posts reversed %}
            
                {% if post.categories contains 'photon-cloud' %}
                {% if post.categories contains cat %}
                    {% if page.title == post.title %}
                        <li class="active">
                            <a href="{{ post.url }}{{ GET }}">{{ post.title }}</a></li>
                    {% else %}
                        <li>
                            <a href="{{ post.url }}{{ GET }}">{{ post.title }}</a></li>
                    {% endif %}
                {% endif %}
                {% endif %}
                
            {% endfor %}
                </ul>
            </li>
            
    {% endfor %}
    </ul>

</aside>
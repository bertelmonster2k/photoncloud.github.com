<aside>
    <nav>
        <ul>
    {% for cat in (page.sidebar_cats) %}
    
            <li>
                <h3>
                    <a href="#">{{ cat | replace:'_',' ' | capitalize }}</a>
                </h3>
                
                <ul class="js-guides">
            {% for post in site.posts reversed %}
            
                {% if post.categories contains page.main_cat %}
                {% if post.categories contains cat %}
                    {% if page.title == post.title %}
                        <li class="disable"><span>{{ page.title }}</span></li>
                    {% else %}
                        <li><a href="{{ post.url }}">{{ post.title }}</a></li>
                    {% endif %}
                {% endif %}
                {% endif %}
                
            {% endfor %}
                </ul>
            </li>
            
    {% endfor %}
        </ul>
    </nav>
</aside>
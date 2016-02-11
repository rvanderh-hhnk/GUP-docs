#########
Home.html
#########

Html “POST”-request gaat via url ‘waypoints-upload’ naar views.py::

	<form enctype="multipart/form-data" method="post" action="{% url 'waypoints-upload' %}">{% csrf_token %}
		<input class='btn btn-primary' type='file' multiple='multiple' name='gef'>
		<input class='btn btn-primary' type='submit' value='Upload GEF'>
	</form>

einde

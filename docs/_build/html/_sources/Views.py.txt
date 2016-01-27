########
Views.py
########

Request.FILES wordt opgeslagen op server (media folder).
Afhandeling gaat per gef-file verder in::

	for gefFile in request.FILES.getlist('gef'):

Dictionary “d_GEF” wordt aangemaakt met functie UtlGefOpen::

	def upload(request):
		if 'gef' in request.FILES['gef']:
			try:
				gefTest = request.FILES['gef']
			except IOError:
				print("gef")
			for gefFile in request.FILES.getlist('gef'):
				files=[]
				handle, targetPath = tempfile.mkstemp()
				destination = os.fdopen(handle, 'wb')
				for chunk in gefFile.chunks():
					destination.write(chunk)
					files.append(targetPath)
			destination.close()
			# 1. ) Dictionary maken van Gef met UtlGefOpen
			try:
				d_GEF = UtlGefOpen.headerdict(targetPath)
			except IOError:
				messages.add_message(request, messages.ERROR,
					'fout bij uitlezen gef, pas %s aan'%gefFile)

2. ) Standaard gef-waarden en 3.) geometrie worden opgehaald met het resultaat d_GEF
In functie "gef_standaard" in **TestGefData.py**::

	# vaste waarden ophalen in TestGefData.py
	try:
		# 2.) standaard gef-waarden ophalen (standaard) = niet-specifiek voor boring, peilbuis of sondering)
		gefFileNaam, CompanyID, ProjectID, StartDatum, SoortGef, X, Y, Z, g = TestGefData.gef_standaard(request,gefFile,d_GEF)
		# 3.) Geometrie punt ophalen
		PointGeom = TestGefData.GetPointGeom(X,Y,Z)
		if request.user.is_authenticated:
			User = str(request.user) # geeft ook AnonymousUser
		else:
			User = "Onbekend"
		# 4.) Opslaan standaard gef-waarden
		if TestGEfData.save_gef(request, d_GEF,gefFile, gefFileNaam, CompanyID, PRojectID, StartDatum, SoortGef, User, X, Y, Z, PointGeom, g) == False:
			raise TypeError
	except TypeError:
		print ("TypeError")
		messages.add_message(request, messages.ERROR, '%s niet kunnen opslaan, controleer bestand'%(gefFile))

4.) variabelen uit stap 2 en 3 gaan naar functie save_gef van **TestGefData.py**   
Als alle gefs zijn opgeslagen wordt de projectdata onderaan bijgewerkt met functie **ProjectCount(request)**::

	# bijwerken van aantal objecten per project
	def ProjectCount(request):
		projecten = Projecten.objects.filter(user_id_id=request.user.id).values_list('project_name', float=True)
		print (str(projecten))
		for project in projecten:
			p = Projecten.opbjects.get(project_name = project).count()
			i_boringen = Boring.objects.filter(project_id=project).count()
			i_peilbuisputten = Peilbuisput.objects.filter(project_id=project).count()
			p.aantal_boringen = i_boringen
			p.aantal_peilbuisputten = i_peilbuisputten
			p.aantal_sonderingen = i_sonderingen
			p.save()
			print ('#boringen:\t%s'%(i_boringen))
			print ("#peilbuisputten:\t%s"%(i_peilbuisputten))
			print ("#sonderingen:\t%s"%(i_sonderingen))


##############
TestGefData.py
##############

De afhandeling in functie **save_gef** van **TestGefData.py** is een van de meer ingewikkeldere onderdelen van het portaal.

Eerst wordt gecheckt of het project van een gef-file al bestaat. Als het project al in gebruik is bij een andere gebruiker wordt de gef afgekeurd met **g = false**. Het script wordt nog wel doorgelezen om te kijken of er nog meer fouten in zitten.

Als het project al bestaat voor de ingelogde gebruiker gebeurd er niets. Als het nog niet bestaat wordt het aangemaakt.::

	# Projecten
	# optie 1) project hoort altijd bij 1 gebruiker
	# check of project al bestaat onder andere gebruiker
	if Projecten.objects.filter(project_name = ProjectID).exists() and not Projecten.objects.filter(project_name = ProjectID, user_id_id = UserId).exists():
		messages.add_message(request, messages.ERROR, 'Project: %s al in gebruik door andere gebruiker'%(ProjectID))
		g = False
	# check of project al bestaat voor huidige gebruiker
	if not Projecten.objects.filter(project_name = ProjectID).exists() and g == True:
		print ("save new project with %s %s "%(ProjectID,UserID))
		project = Projecten(project_name=ProjectID, user_id_id=UserId, username=request.user.username)
		project.save()
	# optie 2) gedeeld projecten: kan verschillende gebruikers hebben. geen constraint
	## if Projecten.objects.filter(project_name = ProjectID, user_id_id = UserId).exists() == False and g == True: # if Projecten.objects.get(id=files_id)

Vanaf dit punt in functie save_gef van TestGefData.py wordt gekeken naar het soort gef-bestand (boring, peilbuisput of sondering) en afhankelijk daarvan behandeld.

====================
Afhandeling boringen
====================

::

	# Boring: nog een keer uitwerken voor andere velden
	if SoortGef == 'boring' and g == True: # boring
		print ("Boring %s bestaat al"%(gefFileNaam))
		b = Boring.objects.get(bestand_gef=gefFileNaam)
		b.bestand_gef=gefFileNaam
		b.download_gef=(hyperlink_root+gefFileNaam)
		b.bedrijf=CompanyID
		b.project_id=ProjectID
		b.datum_boring=StartDatum
		b.username=User
		b.x_rd=X
		b.y_rd=Y
		b.mv_nap=Z
		b.gef_file=gefFile
		b.geometry=PointGeom
		b.save()
		#messages.add_message(request, messages.INFO, '%s: succesvol overschreven!'%GefFile)
	else:
		boring = Boring(bestand_gef=gefFileNaam, download_gef=(hyperlink_root+gefFileNaam), datum_boring=StartDatum, username=User, x_rd=X, y_rd=Y, mv_nap=Z, gef_file=g)
		boring.save()


=======================
Afhandeling sonderingen
=======================

Deze lijkt erg veel op die van boringen::

	if SoortGef == 'sondering' and g == True: # sondering
		if Sondering.objects.filter(bestand_gef=gefFileNaam).exists():
			print ("Sondering %s bestaat al"%(gefFileNaam))
			s = Sondering.objects.get(bestand_gef=gefFileNaam)
			s.bestand_gef=gefFileNaam
			s.download_gef=(hyperlink_root+gefFileNaam)
			s.bedrijf=CompanyId
			s.project_id=ProjectID
			s.datum_sondering=StartDatum
			s.username=User
			s.x_rd=X
			s.y_rd=Y
			s.mv_nap=Z
			s.gef_file=gefFile
			s.geometry=PointGeom
			s.save()
			#messages.add_message(request, messages.INFO, '%s: succesvol overschreven!'%gefFile)
	else: # object bestaat nog niet
		sondering = Sondering(bestand_gef=gefFileNaam, download_gef=(hyperlink_root+gefFileNaam), datum_sondering=StartDatum, username=User, x_rd=X, y_rd=Y, mv_nap=Z, gef_file=g)
		sondering.save()

=====================
Afhandeling waypoints
=====================

Waypoints worden alleen maar gebruikt voor de interactieve tabel op het portaal. Een waypoint kan zowel een boringen, peilbuisput of sondering zijn en bevat alleen algemene informatie (gedefinieerd als gef-standaard oftewel gegevens die niet specifiek zijn en altijd voorkomen ongeacht of het om een boring, peilbuisput of sondering gaat.)::

	# Waypoint
	if Waypoint.objects.filter(name=gefFileNaam).exists() and g == True:
		print ("Waypoint %s bestaat al "%(gefFileNaam))
		w = Waypoint.objects.get(name=gefFIleNaam)
		w.name=gefFileNaam
		w.download_gef=(hyperlink_root+GefFileNaam)
		w.company=CompanyID
		w.projectid=ProjectID
		w.startdatum=StartDatum
		w.username=User
		w.soortgef=SoortGef
		w.x=X
		w.y=Y
		w.z=Z
		w.geometry=PointGeom
		w.save()
		messages.add_message(request, messages.INFO, '%s: succesvol overschreven!'%(gefFile))
	elif g==True:
		waypoint = Waypoint(name=gefFileNaam, download_gef=(hyperlink_root+gefFileNaam), company='x', startdatum=StartDatum, username=User, soortgef=SoortGef, x=X, y=Y, z=Z, geometry=PointGeom)
		waypoint.save()

=======================
Afhandeling peilbuisput
=======================

Peilbuisputten zijn wat ingewikkelder dan de andere objecten. Specifieke attributen worden opgevraagd in functie “gef_peilbuisput” van TestGefData.py::

	# Peilbuisput
	if SoortGef == 'peilbuisput' and g == True:
		# ga naar functie gef_peilbuisput voor specifieke peilbuisput-attributen
		print ("ga hier naar gef_peilbuisput")
		BESTAND_PARENT, BORING_ID, DATUM_PLAATSING, AANTAL_PEILBUIZEN, PROJECTNUMMER = gef_peilbuisput(request,gefFile,d_GEF)
		if BESTAND_PARENT == None or BORING_ID ==None:
			print ("geen parent gevonden")
			messages.add_message(request, messages.ERROR, '%s Parent (boring) bij niet gevonden!'%(gefFile))
			g = False
			return False
		print ("terug uit gef_peilbuisput")
		# print (Peilbuisput.objects.get(bestand_gef=gefFileNaam))
		if Peilbuisput.objects.filter(bestand_gef=gefFileNaam).exists() and g==True:
			print ("Peilbuisput %s bestaat al"%(gefFileNaam))
			p = Peilbuisput.objects.get(bestand_gef=gefFileNaam)
			print (p)
			p.status = "test"
			p.boreholeident = None
			p.boring_id_id = BORING_ID
			p.peilbuisraai_id = None
			p.x_rd = X
			p.y_rd = Yz
			p.dwarspositie = None
			p.datum_plaatsing = StartDatum #DATUM_PLAATSING moet nog omgezet worden naar 'yymmdd'
			p.aanwezig = None
			p.datum_verwijdering = None
			p.bedrijf = CompanyID
			p.project_id = ProjectID
			p.type_peilbuizen = None
			p.aantal_peilbuizen = AANTAL_PEILBUIZEN
			p.bestand_peilbuisput = None
			p.bestand_parent = BESTAND_PARENT
			p.gtlp_id = None
			p.bestand_gef = gefFileNaam
			p.bestand_grondonderzoek = None
			p.gef_file = gefFile
			p.download_gef = (hyperlink_root+gefFileNaam)
			# status
			# #DateCreated =
			# #DateMutated =
			p.username = User
			p.geometry = PointGeom
			p.save()
			#messages.add_message(request, messages.INFO, '%s: succesvol overschreven!'%(gefFile))
		elif g == True:
			peilbuisput = Peilbuisput(boreholeident=None)

=======================
Functie gef_peilbuisput
=======================
Voert tests uit voor de correctheid van de peilbuisput. Haalt specifieke data op zoals het BORING_ID.::

	def gef_peilbuisput(request,gefFile,d_GEF):
		#'''	haalt de standaard gegevens op uit gef-bestanden via UtlGefOpen.
		#	functie gef_peilbuisput voor specifieke peilbuisput-attributen
		#	voert verschillende checks uit en genereert desgewenst foutmeldingen.'''
		# checks werken nog niet, dus alles op 'false'
		sTEST1 = 'false' # plotable
		sTEST2 = 'false' # consistent data block
		sTEST3 = 'false' # correcte header
		sTEST4 = 'false' # beschreven als BOREHOLE-Report
		# Valideer het ingelezen bestand eerst
		print ("gaat naar IsCorrectBestandPeilbuisput")
		if not isCorrectBestandPeilbuisput(gefFile, d_GEF, sTEST1, sTEST2, sTEST3, sTEST4):
			# Bestand is niet in orde, skip
			# pass
			return None, None, None, None, None
		else:
			print ("gaat verder na IsCOrrectBestandPeilbuisput")
			# Attribuutwaarden
			# iMaxID +=1 # verhoog volgnummer ID
			# sPeilbuisputID = sFORMATSTRING % iMaxID, b.k.note: is dit nodig om zelf hier te bepalen
			# BOREHOLE_ID = sPeilbuisputID
			# BESTAND_PEILBUISPUT = sGefBestand # bestandsnaam, b.k.note: bestandsnaam weten we al.
			sBestandParent = UtlGefOpen.Get_PBP_BestandParent(d_GEF)
			BESTAND_PARENT = sBestandParent
			BORING_ID = GetBoringID(BESTAND_PARENT)
			print("BORING_ID = %s"%(BORING_ID))
			if not BORING_ID == None:
				BORING_ID = int(BORING_ID)
			PROJECTNUMMER = UtlGefOpen.Get_ALG_ProjectNummer(d_GEF)
			DATUM_PLAATSING = UtlGefOpen.Get_PBP_DatumPlaatsing(d_GEF)
			AANTAL_PEILBUIZEN = UtlGefOpen.Get_PBP_AantalPeilbuizen(d_GEF)
			print(PROJECTNUMMER,DATUM,PLAATSING,AANTAL_PEILBUIZEN)
			# VUl de bijbehorende peilbuisgegevens
			# VulTblPb(i_oGp, i_sPadTargetTblPb, sPeilbuisputID, i_sPadTargetTblFt)
			# iCountPunten += 1
			return BESTAND_PARENT, BORING_ID, DATUM_PLAATSING, AANTAL_PEILBUIZEN, PROJECTNUMMER

		# Print ('\nTotaal aantal GEF bestanden in folder: %6i' %(len(oListGefBestanden)))
		# Print ('Aantal peilbuisput locaties toegevoegd: %6i' %(iCountPunten))
		# Print ('- met daarin aantal peilbuizen: %6i'%(iCountPb))
		# Print ('- met daarin aantal filters: %6i'%(iCountFt))

::

	# -------------------------------------------------------------------------------
	# Purpose: Haal uit fc BORING het ID dat hoort bij de opgegeven parent-
	#	   bestandsnaam
	# -------------------------------------------------------------------------------
	def GetBoringID(BESTAND_PARENT):
		# Maxmimale afstand tussen peilbuisput en parent-boring
		fMAX_AFSTAND = 10.0
		sResult = None
		iCount = 0
		print ("zoekt "+str(BESTAND_PARENT)+" in Boringen.bestand_gef")
		q = Boring.objects.all().filter(bestand_gef = BESTAND_PARENT)
		if len(q)>0:
			print ("%i boringen gevonden met %s"%(len(q),BESTAND_PARENT))
			q = q[0]
			sResult = q.id
		return sResult

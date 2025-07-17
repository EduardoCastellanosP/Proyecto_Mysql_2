

#  Proyecto de Base de Datos: Plataforma de Comercialización Digital Multinivel



#### CREACION DE TABLAS

```SQL
CREATE TABLE countries(
  isocode INTEGER PRIMARY KEY,
  name VARCHAR(50) UNIQUE,
  alfatwocode VARCHAR(2) UNIQUE,
  alfathreecode VARCHAR(4) UNIQUE
);

CREATE TABLE subdivisioncategories(
  id VARCHAR(20) PRIMARY KEY,
  description VARCHAR(40)
);

CREATE TABLE typeidentifications(
  id INT PRIMARY KEY AUTO_INCREMENT,
  description VARCHAR(60) UNIQUE,
  suffix VARCHAR(5)
);

CREATE TABLE audiences(
  id INT PRIMARY KEY AUTO_INCREMENT,
  description VARCHAR(60) UNIQUE
);

CREATE TABLE categories(
  id INT PRIMARY KEY AUTO_INCREMENT,
  description VARCHAR(60) UNIQUE
);

CREATE TABLE unitofmeasure(
  id INT PRIMARY KEY AUTO_INCREMENT,
  description VARCHAR(60) UNIQUE
);

CREATE TABLE categories_polls(
  id INT PRIMARY KEY AUTO_INCREMENT,
  name VARCHAR(80) UNIQUE
);

CREATE TABLE memberships(
  id INT PRIMARY KEY AUTO_INCREMENT,
  name VARCHAR(50) UNIQUE,
  description TEXT
);

CREATE TABLE periods(
  id INT PRIMARY KEY AUTO_INCREMENT,
  name VARCHAR(50) UNIQUE
);

CREATE TABLE benefits(
  id INT PRIMARY KEY AUTO_INCREMENT,
  description VARCHAR(80),
  detail TEXT
);

CREATE TABLE stateregions(
  code VARCHAR(6) PRIMARY KEY,
  name VARCHAR(50),
  country_id INTEGER,
  code3166 VARCHAR(10),
  subdivision_id VARCHAR(20),
  FOREIGN KEY (country_id) REFERENCES countries(isocode),
  FOREIGN KEY (subdivision_id) REFERENCES subdivisioncategories(id)
);

CREATE TABLE citiesormunicipalities(
  code VARCHAR(6) PRIMARY KEY,
  name VARCHAR(50) UNIQUE,
  statereg_id VARCHAR(6),
  FOREIGN KEY (statereg_id) REFERENCES stateregions(code)
);

CREATE TABLE companies(
  id INT AUTO_INCREMENT PRIMARY KEY,
  name VARCHAR(80),
  type_id INT,
  category_id INT,
  city_id VARCHAR(6),
  audience_id INT,
  FOREIGN KEY (type_id) REFERENCES typeidentifications(id),
  FOREIGN KEY (category_id) REFERENCES categories(id),
  FOREIGN KEY (city_id) REFERENCES citiesormunicipalities(code),
  FOREIGN KEY (audience_id) REFERENCES audiences(id)
);

CREATE TABLE company_phones(
  company_id INT,
  phone VARCHAR(15),
  PRIMARY KEY (company_id, phone),
  FOREIGN KEY (company_id) REFERENCES companies(id)
);

CREATE TABLE company_emails(
  company_id INT,
  email VARCHAR(80),
  PRIMARY KEY (company_id, email),
  FOREIGN KEY (company_id) REFERENCES companies(id)
);

CREATE TABLE customers(
  id INT PRIMARY KEY AUTO_INCREMENT,
  name VARCHAR(60),
  city_id VARCHAR(6),
  audience_id INT,
  FOREIGN KEY (city_id) REFERENCES citiesormunicipalities(code),
  FOREIGN KEY (audience_id) REFERENCES audiences(id)
);

CREATE TABLE customer_phones(
  customer_id INT,
  phone VARCHAR(20),
  PRIMARY KEY (customer_id, phone),
  FOREIGN KEY (customer_id) REFERENCES customers(id)
);

CREATE TABLE customer_emails(
  customer_id INT,
  email VARCHAR(100),
  PRIMARY KEY (customer_id, email),
  FOREIGN KEY (customer_id) REFERENCES customers(id)
);

CREATE TABLE addresses(
  id INT PRIMARY KEY AUTO_INCREMENT,
  street VARCHAR(80),
  city_id VARCHAR(6),
  postal_code VARCHAR(10),
  FOREIGN KEY (city_id) REFERENCES citiesormunicipalities(code)
);

CREATE TABLE customer_addresses(
  customer_id INT,
  address_id INT,
  PRIMARY KEY (customer_id, address_id),
  FOREIGN KEY (customer_id) REFERENCES customers(id),
  FOREIGN KEY (address_id) REFERENCES addresses(id)
);

CREATE TABLE products(
  id INT PRIMARY KEY AUTO_INCREMENT,
  name VARCHAR(80),
  detail TEXT,
  price DOUBLE,
  category_id INT,
  image VARCHAR(80),
  FOREIGN KEY (category_id) REFERENCES categories(id)
);

CREATE TABLE companyproducts(
  company_id INT,
  product_id INT,
  unitmeasure_id INT,
  PRIMARY KEY (company_id, product_id),
  FOREIGN KEY (company_id) REFERENCES companies(id),
  FOREIGN KEY (product_id) REFERENCES products(id),
  FOREIGN KEY (unitmeasure_id) REFERENCES unitofmeasure(id)
);

CREATE TABLE companyproduct_prices(
  company_id INT,
  product_id INT,
  price DOUBLE,
  start_date DATETIME,
  end_date DATETIME,
  PRIMARY KEY (company_id, product_id, start_date),
  FOREIGN KEY (company_id, product_id) REFERENCES companyproducts(company_id, product_id)
);

CREATE TABLE favorites(
  id INT PRIMARY KEY AUTO_INCREMENT,
  customer_id INT,
  company_id INT,
  FOREIGN KEY (customer_id) REFERENCES customers(id),
  FOREIGN KEY (company_id) REFERENCES companies(id)
);

CREATE TABLE details_favorites(
  favorite_id INT,
  product_id INT,
  category_id INT,
  name VARCHAR(80),
  detail TEXT,
  PRIMARY KEY (favorite_id, product_id),
  FOREIGN KEY (favorite_id) REFERENCES favorites(id),
  FOREIGN KEY (product_id) REFERENCES products(id)
);

CREATE TABLE polls(
  id INT PRIMARY KEY AUTO_INCREMENT,
  name VARCHAR(80) UNIQUE,
  description TEXT,
  isactive BOOLEAN,
  categorypoll_id INT,
  FOREIGN KEY (categorypoll_id) REFERENCES categories_polls(id)
);

CREATE TABLE rates(
  customer_id INT,
  company_id INT,
  poll_id INT,
  daterating DATETIME,
  rating DOUBLE,
  PRIMARY KEY (customer_id, company_id, poll_id),
  FOREIGN KEY (customer_id) REFERENCES customers(id),
  FOREIGN KEY (company_id) REFERENCES companies(id),
  FOREIGN KEY (poll_id) REFERENCES polls(id)
);

CREATE TABLE quality_products(
  product_id INT,
  customer_id INT,
  poll_id INT,
  company_id INT,
  daterating DATETIME,
  rating DOUBLE,
  PRIMARY KEY (product_id, customer_id, poll_id, company_id),
  FOREIGN KEY (product_id) REFERENCES products(id),
  FOREIGN KEY (customer_id) REFERENCES customers(id),
  FOREIGN KEY (poll_id) REFERENCES polls(id),
  FOREIGN KEY (company_id) REFERENCES companies(id)
);

CREATE TABLE membershipperiods(
  membership_id INT,
  period_id INT,
  price DOUBLE,
  PRIMARY KEY (membership_id, period_id),
  FOREIGN KEY (membership_id) REFERENCES memberships(id),
  FOREIGN KEY (period_id) REFERENCES periods(id)
);

CREATE TABLE membershipbenefits(
  membership_id INT,
  period_id INT,
  audience_id INT,
  benefit_id INT,
  PRIMARY KEY (membership_id, period_id, audience_id, benefit_id),
  FOREIGN KEY (membership_id) REFERENCES memberships(id),
  FOREIGN KEY (period_id) REFERENCES periods(id),
  FOREIGN KEY (audience_id) REFERENCES audiences(id),
  FOREIGN KEY (benefit_id) REFERENCES benefits(id)
```

Se normalizó lo mas que se pudo, se agregaron unas tablas adicionales para dar mas orden a la hora de buscar en ellas.

#### **INSERTS** 

##### COLOMBIA

```sql
INSERT INTO countries(isocode, name, alfatwocode, alfathreecode) VALUES (170,'colombia','CO','COL'),
(840,'estados unidos','US','USA');

INSERT INTO stateregions (code, name, country_id, code_3166_2, subdivision_id) VALUES 
('91', 'Amazonas', 170, 'CO-AMA', 1),
('05', 'Antioquia', 170, 'CO-ANT', 1),
('81', 'Arauca', 170, 'CO-ARA', 1),
('08', 'Atlántico', 170, 'CO-ATL', 1),
('13', 'Bolívar', 170, 'CO-BOL', 1),
('15', 'Boyacá', 170, 'CO-BOY', 1),
('17', 'Caldas', 170, 'CO-CAL', 1),
('18', 'Caquetá', 170, 'CO-CAQ', 1),
('85', 'Casanare', 170, 'CO-CAS', 1),
('19', 'Cauca', 170, 'CO-CAU', 1),
('20', 'Cesar', 170, 'CO-CES', 1),
('27', 'Chocó', 170, 'CO-CHO', 1),
('25', 'Cundinamarca', 170, 'CO-CUN', 1),
('23', 'Córdoba', 170, 'CO-COR', 1),
('11', 'Distrito Capital de Bogotá', 170, 'CO-DC', 2),
('94', 'Guainía', 170, 'CO-GUA', 1),
('95', 'Guaviare', 170, 'CO-GUV', 1),
('41', 'Huila', 170, 'CO-HUI', 1),
('44', 'La Guajira', 170, 'CO-LAG', 1),
('47', 'Magdalena', 170, 'CO-MAG', 1),
('50', 'Meta', 170, 'CO-MET', 1),
('52', 'Nariño', 170, 'CO-NAR', 1),
('54', 'Norte de Santander', 170, 'CO-NSA', 1),
('86', 'Putumayo', 170, 'CO-PUT', 1),
('63', 'Quindío', 170, 'CO-QUI', 1),
('66', 'Risaralda', 170, 'CO-RIS', 1),
('88', 'San Andrés, Providencia y Santa Catalina', 170, 'CO-SAP', 1),
('68', 'Santander', 170, 'CO-SAN', 1),
('70', 'Sucre', 170, 'CO-SUC', 1),
('73', 'Tolima', 170, 'CO-TOL', 1),
('76', 'Valle del Cauca', 170, 'CO-VAC', 1),
('97', 'Vaupés', 170, 'CO-VAU', 1),
('99', 'Vichada', 170, 'CO-VID', 1);

INSERT INTO subdivisioncategories (id, description) VALUES 
(1,'departament'),(2,'capital district'),(3,'state'),(4,'outliying area');

INSERT INTO citiesormunicipalities (code, name, statereg_id) VALUES
('05001', 'MEDELLIN', '05'),
('05002', 'ABEJORRAL', '05'),
('05004', 'ABRIAQUI', '05'),
('05021', 'ALEJANDRIA', '05'),
('05030', 'AMAGA', '05'),
('05031', 'AMALFI', '05'),
('05034', 'ANDES', '05'),
('05036', 'ANGELOPOLIS', '05'),
('05038', 'ANGOSTURA', '05'),
('05040', 'ANORI', '05'),
('05042', 'SANTAFE DE ANTIOQUIA', '05'),
('05044', 'ANZA', '05'),
('05045', 'APARTADO', '05'),
('05051', 'ARBOLETES', '05'),
('05055', 'ARGELIA', '05'),
('05059', 'ARMENIA', '05'),
('05079', 'BARBOSA', '05'),
('05086', 'BELMIRA', '05'),
('05088', 'BELLO', '05'),
('05091', 'BETANIA', '05'),
('05093', 'BETULIA', '05'),
('05101', 'CIUDAD BOLIVAR', '05'),
('05107', 'BRICEÑO', '05'),
('05113', 'BURITICA', '05'),
('05120', 'CACERES', '05'),
('05125', 'CAICEDO', '05'),
('05129', 'CALDAS', '05'),
('05134', 'CAMPAMENTO', '05'),
('05138', 'CAÑASGORDAS', '05'),
('05142', 'CARACOLI', '05'),
('05145', 'CARAMANTA', '05'),
('05147', 'CAREPA', '05'),
('05148', 'EL CARMEN DE VIBORAL', '05'),
('05150', 'CAROLINA', '05'),
('05154', 'CAUCASIA', '05'),
('05172', 'CHIGORODO', '05'),
('05190', 'CISNEROS', '05'),
('05197', 'COCORNA', '05'),
('05206', 'CONCEPCION', '05'),
('05209', 'CONCORDIA', '05'),
('05212', 'COPACABANA', '05'),
('05234', 'DABEIBA', '05'),
('05237', 'DON MATIAS', '05'),
('05240', 'EBEJICO', '05'),
('05250', 'EL BAGRE', '05'),
('05264', 'ENTRERRIOS', '05'),
('05266', 'ENVIGADO', '05'),
('05282', 'FREDONIA', '05'),
('05284', 'FRONTINO', '05'),
('05306', 'GIRALDO', '05'),
('05308', 'GIRARDOTA', '05'),
('05310', 'GOMEZ PLATA', '05'),
('05313', 'GRANADA', '05'),
('05315', 'GUADALUPE', '05'),
('05318', 'GUARNE', '05'),
('05321', 'GUATAPE', '05'),
('05347', 'HELICONIA', '05'),
('05353', 'HISPANIA', '05'),
('05360', 'ITAGÜI', '05'),
('05361', 'ITUANGO', '05'),
('05364', 'JARDIN', '05'),
('05368', 'JERICO', '05'),
('05376', 'LA CEJA', '05'),
('05380', 'LA ESTRELLA', '05'),
('05390', 'LA PINTADA', '05'),
('05400', 'LA UNION', '05'),
('05411', 'LIBORINA', '05'),
('05425', 'MACEO', '05'),
('05440', 'MARINILLA', '05'),
('05467', 'MONTEBELLO', '05'),
('05475', 'MURINDO', '05'),
('05480', 'MUTATA', '05'),
('05483', 'NARIÑO', '05'),
('05490', 'NECOCLI', '05'),
('05495', 'NECHI', '05'),
('05501', 'OLAYA', '05'),
('05541', 'PEÑOL', '05'),
('05543', 'PEQUE', '05'),
('05576', 'PUEBLORRICO', '05'),
('05579', 'PUERTO BERRIO', '05'),
('05585', 'PUERTO NARE', '05'),
('05591', 'PUERTO TRIUNFO', '05'),
('05604', 'REMEDIOS', '05'),
('05607', 'RETIRO', '05'),
('05615', 'RIONEGRO', '05'),
('05628', 'SABANALARGA', '05'),
('05631', 'SABANETA', '05'),
('05642', 'SALGAR', '05'),
('05647', 'SAN ANDRES', '05'),
('05649', 'SAN CARLOS', '05'),
('05652', 'SAN FRANCISCO', '05'),
('05656', 'SAN JERONIMO', '05'),
('05658', 'SAN JOSE DE LA MONTAÑA', '05'),
('05659', 'SAN JUAN DE URABA', '05'),
('05660', 'SAN LUIS', '05'),
('05664', 'SAN PEDRO', '05'),
('05665', 'SAN PEDRO DE URABA', '05'),
('05667', 'SAN RAFAEL', '05'),
('05670', 'SAN ROQUE', '05'),
('05674', 'SAN VICENTE', '05'),
('05679', 'SANTA BARBARA', '05'),
('05686', 'SANTA ROSA DE OSOS', '05'),
('05690', 'SANTO DOMINGO', '05'),
('05697', 'EL SANTUARIO', '05'),
('05736', 'SEGOVIA', '05'),
('05756', 'SONSON', '05'),
('05761', 'SOPETRAN', '05'),
('05789', 'TAMESIS', '05'),
('05790', 'TARAZA', '05'),
('05792', 'TARSO', '05'),
('05809', 'TITIRIBI', '05'),
('05819', 'TOLEDO', '05'),
('05837', 'TURBO', '05'),
('05842', 'URAMITA', '05'),
('05847', 'URRAO', '05'),
('05854', 'VALDIVIA', '05'),
('05856', 'VALPARAISO', '05'),
('05858', 'VEGACHI', '05'),
('05861', 'VENECIA', '05'),
('05873', 'VIGIA DEL FUERTE', '05'),
('05885', 'YALI', '05'),
('05887', 'YARUMAL', '05'),
('05890', 'YOLOMBO', '05'),
('05893', 'YONDO', '05'),
('05895', 'ZARAGOZA', '05');


```

```sql
INSERT INTO citiesormunicipalities (code, name, statereg_id) VALUES
('08001', 'BARRANQUILLA', '08'),
('08078', 'BARANOA', '08'),
('08137', 'CAMPO DE LA CRUZ', '08'),
('08141', 'CANDELARIA', '08'),
('08296', 'GALAPA', '08'),
('0372', 'JUAN DE ACOSTA', '08'),
('08421', 'LURUACO', '08'),
('08433', 'MALAMBO', '08'),
('08436', 'MANATI', '08'),
('08520', 'PALMAR DE VARELA', '08'),
('08549', 'PIOJO', '08'),
('08558', 'POLONUEVO', '08'),
('08560', 'PONEDERA', '08'),
('08573', 'PUERTO COLOMBIA', '08'),
('08606', 'REPELON', '08'),
('08634', 'SABANAGRANDE','08'),
('08638', 'SABANALARGA','08'),
('08675', 'SANTA LUCIA', '08'),
('08685', 'SANTO TOMAS', '08'),
('08758', 'SOLEDAD', '08'),
('08770', 'SUAN', '08'),
('08832', 'TUBARA','08'),
('08849', 'USIACURI', '08');

```

```sql
INSERT INTO citiesormunicipalities (code, name, statereg_id) VALUES
('13001', 'CARTAGENA', '13'),
('13006', 'ACHI', '13'),
('13030', 'ALTOS DEL ROSARIO', '13'),
('13042', 'ARENAL', '13'),
('13052', 'ARJONA', '13'),
('13062', 'ARROYOHONDO', '13'),
('13074', 'BARRANCO DE LOBA', '13'),
('13140', 'CALAMAR', '13'),
('13160', 'CANTAGALLO', '13'),
('13188', 'CICUCO', '13'),
('13212', 'CORDOBA', '13'),
('13222', 'CLEMENCIA', '13'),
('13244', 'EL CARMEN DE BOLIVAR', '13'),
('13248', 'EL GUAMO', '13'),
('13268', 'EL PEÑON', '13'),
('13300', 'HATILLO DE LOBA', '13'),
('13430', 'MAGANGUE', '13'),
('13433', 'MAHATES', '13'),
('13440', 'MARGARITA', '13'),
('13442', 'MARIA LA BAJA', '13'),
('13458', 'MONTECRISTO', '13'),
('13468', 'MOMPOS', '13'),
('13473', 'MORALES', '13'),
('13549', 'PINILLOS', '13'),
('13580', 'REGIDOR', '13'),
('13600', 'RIO VIEJO', '13'),
('13620', 'SAN CRISTOBAL', '13'),
('13647', 'SAN ESTANISLAO', '13'),
('13650', 'SAN FERNANDO', '13'),
('13654', 'SAN JACINTO', '13'),
('13655', 'SAN JACINTO DEL CAUCA', '13'),
('13657', 'SAN JUAN NEPOMUCENO', '13'),
('13667', 'SAN MARTIN DE LOBA', '13'),
('13670', 'SAN PABLO', '13'),
('13673', 'SANTA CATALINA', '13'),
('13683', 'SANTA ROSA', '13'),
('13688', 'SANTA ROSA DEL SUR', '13'),
('13744', 'SIMITI', '13'),
('13760', 'SOPLAVIENTO', '13'),
('13780', 'TALAIGUA NUEVO', '13'),
('13810', 'TIQUISIO', '13'),
('13836', 'TURBACO', '13'),
('13838', 'TURBANA', '13'),
('13873', 'VILLANUEVA', '13'),
('13894', 'ZAMBRANO', '13');


```

```sql
INSERT INTO citiesormunicipalities (code, name, statereg_id) VALUES
('15001', 'TUNJA', '15'),
('15022', 'ALMEIDA', '15'),
('15047', 'AQUITANIA', '15'),
('15051', 'ARCABUCO', '15'),
('15087', 'BELEN', '15'),
('15090', 'BERBEO', '15'),
('15092', 'BETEITIVA', '15'),
('15097', 'BOAVITA', '15'),
('15104', 'BOYACA', '15'),
('15106', 'BRICEÑO', '15'),
('15109', 'BUENAVISTA', '15'),
('15114', 'BUSBANZA', '15'),
('15131', 'CALDAS', '15'),
('15135', 'CAMPOHERMOSO', '15'),
('15162', 'CERINZA', '15'),
('15172', 'CHINAVITA', '15'),
('15176', 'CHIQUINQUIRA', '15'),
('15180', 'CHISCAS', '15'),
('15183', 'CHITA', '15'),
('15185', 'CHITARAQUE', '15'),
('15187', 'CHIVATA', '15'),
('15189', 'CIENEGA', '15'),
('15204', 'COMBITA', '15'),
('15212', 'COPER', '15'),
('15215', 'CORRALES', '15'),
('15218', 'COVARACHIA', '15'),
('15223', 'CUBARA', '15'),
('15224', 'CUCAITA', '15'),
('15226', 'CUITIVA', '15'),
('15232', 'CHIQUIZA', '15'),
('15236', 'CHIVOR', '15'),
('15238', 'DUITAMA', '15'),
('15244', 'EL COCUY', '15'),
('15248', 'EL ESPINO', '15'),
('15272', 'FIRAVITOBA', '15'),
('15276', 'FLORESTA', '15'),
('15293', 'GACHANTIVA', '15'),
('15296', 'GAMEZA', '15'),
('15299', 'GARAGOA', '15'),
('15317', 'GUACAMAYAS', '15'),
('15322', 'GUATEQUE', '15'),
('15325', 'GUAYATA', '15'),
('15332', 'GÜICAN', '15'),
('15362', 'IZA', '15'),
('15367', 'JENESANO', '15'),
('15368', 'JERICO', '15'),
('15377', 'LABRANZAGRANDE', '15'),
('15380', 'LA CAPILLA', '15'),
('15401', 'LA VICTORIA', '15'),
('15403', 'LA UVITA', '15'),
('15407', 'VILLA DE LEYVA', '15'),
('15425', 'MACANAL', '15'),
('15442', 'MARIPI', '15'),
('15455', 'MIRAFLORES', '15'),
('15464', 'MONGUA', '15'),
('15466', 'MONGUI', '15'),
('15469', 'MONIQUIRA', '15'),
('15476', 'MOTAVITA', '15'),
('15480', 'MUZO', '15'),
('15491', 'NOBSA', '15'),
('15494', 'NUEVO COLON', '15'),
('15500', 'OICATA', '15'),
('15507', 'OTANCHE', '15'),
('15511', 'PACHAVITA', '15'),
('15514', 'PAEZ', '15'),
('15516', 'PAIPA', '15'),
('15518', 'PAJARITO', '15'),
('15522', 'PANQUEBA', '15'),
('15531', 'PAUNA', '15'),
('15533', 'PAYA', '15'),
('15537', 'PAZ DE RIO', '15'),
('15542', 'PESCA', '15'),
('15550', 'PISBA', '15'),
('15572', 'PUERTO BOYACA', '15'),
('15580', 'QUIPAMA', '15'),
('15599', 'RAMIRIQUI', '15'),
('15600', 'RAQUIRA', '15'),
('15621', 'RONDON', '15'),
('15632', 'SABOYA', '15'),
('15638', 'SACHICA', '15'),
('15646', 'SAMACA', '15'),
('15660', 'SAN EDUARDO', '15'),
('15664', 'SAN JOSE DE PARE', '15'),
('15667', 'SAN LUIS DE GACENO', '15'),
('15673', 'SAN MATEO', '15'),
('15676', 'SAN MIGUEL DE SEMA', '15'),
('15681', 'SAN PABLO DE BORBUR', '15'),
('15686', 'SANTANA', '15'),
('15690', 'SANTA MARIA', '15'),
('15693', 'SANTA ROSA DE VITERBO', '15'),
('15696', 'SANTA SOFIA', '15'),
('15720', 'SATIVANORTE', '15'),
('15723', 'SATIVASUR', '15'),
('15740', 'SIACHOQUE', '15'),
('15753', 'SOATA', '15'),
('15755', 'SOCOTA', '15'),
('15757', 'SOCHA', '15'),
('15759', 'SOGAMOSO', '15'),
('15761', 'SOMONDOCO', '15'),
('15762', 'SORA', '15'),
('15763', 'SOTAQUIRA', '15'),
('15764', 'SORACA', '15'),
('15774', 'SUSACON', '15'),
('15776', 'SUTAMARCHAN', '15'),
('15778', 'SUTATENZA', '15'),
('15790', 'TASCO', '15'),
('15798', 'TENZA', '15'),
('15804', 'TIBANA', '15'),
('15806', 'TIBASOSA', '15'),
('15808', 'TINJACA', '15'),
('15810', 'TIPACOQUE', '15'),
('15814', 'TOCA', '15'),
('15816', 'TOGÜI', '15'),
('15820', 'TOPAGA', '15'),
('15822', 'TOTA', '15'),
('15832', 'TUNUNGUA', '15'),
('15835', 'TURMEQUE', '15'),
('15837', 'TUTA', '15'),
('15839', 'TUTAZA', '15'),
('15842', 'UMBITA', '15'),
('15861', 'VENTAQUEMADA', '15'),
('15879', 'VIRACACHA', '15'),
('15897', 'ZETAQUIRA', '15');

```

```sql
INSERT INTO citiesormunicipalities (code, name, statereg_id) VALUES
('17001', 'MANIZALES', '17'),
('17013', 'AGUADAS', '17'),
('17042', 'ANSERMA', '17'),
('17050', 'ARANZAZU', '17'),
('17088', 'BELALCAZAR', '17'),
('17174', 'CHINCHINA', '17'),
('17272', 'FILADELFIA', '17'),
('17380', 'LA DORADA', '17'),
('17388', 'LA MERCED', '17'),
('17433', 'MANZANARES', '17'),
('17442', 'MARMATO', '17'),
('17444', 'MARQUETALIA', '17'),
('17446', 'MARULANDA', '17'),
('17486', 'NEIRA', '17'),
('17495', 'NORCASIA', '17'),
('17513', 'PACORA', '17'),
('17524', 'PALESTINA', '17'),
('17541', 'PENSILVANIA', '17'),
('17614', 'RIOSUCIO', '17'),
('17616', 'RISARALDA', '17'),
('17653', 'SALAMINA', '17'),
('17662', 'SAMANA', '17'),
('17665', 'SAN JOSE', '17'),
('17777', 'SUPIA', '17'),
('17867', 'VICTORIA', '17'),
('17873', 'VILLAMARIA', '17'),
('17877', 'VITERBO', '17');

```

```sql
INSERT INTO citiesormunicipalities (code, name, statereg_id) VALUES
('18001', 'FLORENCIA', 18),
('18029', 'ALBANIA', 18),
('18094', 'BELEN DE LOS ANDAQUIES', 18),
('18150', 'CARTAGENA DEL CHAIRA', 18),
('18205', 'CURILLO', 18),
('18247', 'EL DONCELLO', 18),
('18256', 'EL PAUJIL', 18),
('18410', 'LA MONTAÑITA', 18),
('18460', 'MILAN', 18),
('18479', 'MORELIA', 18),
('18592', 'PUERTO RICO', 18),
('18610', 'SAN JOSE DE LA FRAGUA', 18),
('18753', 'SAN VICENTE DEL CAGUAN', 18),
('18756', 'SOLANO', 18),
('18785', 'SOLITA', 18),
('18860', 'VALPARAISO', 18);

```

```sql
INSERT INTO citiesormunicipalities (code, name, statereg_id) VALUES
('19001', 'POPAYAN', '19'),
('19022', 'ALMAGUER', '19'),
('19050', 'ARGELIA', '19'),
('19075', 'BALBOA', '19'),
('19100', 'BOLIVAR', '19'),
('19110', 'BUENOS AIRES', '19'),
('19130', 'CAJIBIO', '19'),
('19137', 'CALDONO', '19'),
('19142', 'CALOTO', '19'),
('19212', 'CORINTO', '19'),
('19256', 'EL TAMBO', '19'),
('19290', 'FLORENCIA', '19'),
('19318', 'GUAPI', '19'),
('19355', 'INZA', '19'),
('19364', 'JAMBALO', '19'),
('19392', 'LA SIERRA', '19'),
('19397', 'LA VEGA', '19'),
('19418', 'LOPEZ', '19'),
('19450', 'MERCADERES', '19'),
('19455', 'MIRANDA', '19'),
('19473', 'MORALES', '19'),
('19513', 'PADILLA', '19'),
('19517', 'PAEZ', '19'),
('19532', 'PATIA', '19'),
('19533', 'PIAMONTE', '19'),
('19548', 'PIENDAMO', '19'),
('19573', 'PUERTO TEJADA', '19'),
('19585', 'PURACE', '19'),
('19622', 'ROSAS', '19'),
('19693', 'SAN SEBASTIAN', '19'),
('19698', 'SANTANDER DE QUILICHAO', '19'),
('19701', 'SANTA ROSA', '19'),
('19743', 'SILVIA', '19'),
('19760', 'SOTARA', '19'),
('19780', 'SUAREZ', '19'),
('19785', 'SUCRE', '19'),
('19807', 'TIMBIO', '19'),
('19809', 'TIMBIQUI', '19'),
('19821', 'TORIBIO', '19'),
('19824', 'TOTORO', '19'),
('19845', 'VILLA RICA', '19');
```

```sql
INSERT INTO citiesormunicipalities (code, name, statereg_id) VALUES
('20001', 'VALLEDUPAR', '20'),
('20011', 'AGUACHICA', '20'),
('20013', 'AGUSTIN CODAZZI', '20'),
('20032', 'ASTREA', '20'),
('20045', 'BECERRIL', '20'),
('20060', 'BOSCONIA', '20'),
('20175', 'CHIMICHAGUA', '20'),
('20178', 'CHIRIGUANA', '20'),
('20228', 'CURUMANI', '20'),
('20238', 'EL COPEY', '20'),
('20250', 'EL PASO', '20'),
('20295', 'GAMARRA', '20'),
('20310', 'GONZALEZ', '20'),
('20383', 'LA GLORIA', '20'),
('20400', 'LA JAGUA DE IBIRICO', '20'),
('20443', 'MANAURE', '20'),
('20517', 'PAILITAS', '20'),
('20550', 'PELAYA', '20'),
('20570', 'PUEBLO BELLO', '20'),
('20614', 'RIO DE ORO', '20'),
('20621', 'LA PAZ', '20'),
('20710', 'SAN ALBERTO', '20'),
('20750', 'SAN DIEGO', '20'),
('20770', 'SAN MARTIN', '20'),
('20787', 'TAMALAMEQUE', '20');

```

```sql
INSERT INTO citiesormunicipalities (code, name, statereg_id) VALUES
('23001', 'MONTERIA', '23'),
('23068', 'AYAPEL', '23'),
('23079', 'BUENAVISTA', '23'),
('23090', 'CANALETE', '23'),
('23162', 'CERETE', '23'),
('23168', 'CHIMA', '23'),
('23182', 'CHINU', '23'),
('23189', 'CIENAGA DE ORO', '23'),
('23300', 'COTORRA', '23'),
('23350', 'LA APARTADA', '23'),
('23417', 'LORICA', '23'),
('23419', 'LOS CORDOBAS', '23'),
('23464', 'MOMIL', '23'),
('23466', 'MONTELIBANO', '23'),
('23500', 'MOÑITOS', '23'),
('23555', 'PLANETA RICA', '23'),
('23570', 'PUEBLO NUEVO', '23'),
('23574', 'PUERTO ESCONDIDO', '23'),
('23580', 'PUERTO LIBERTADOR', '23'),
('23586', 'PURISIMA', '23'),
('23660', 'SAHAGUN', '23'),
('23670', 'SAN ANDRES DE SOTAVENTO', '23'),
('23672', 'SAN ANTERO', '23'),
('23675', 'SAN BERNARDO DEL VIENTO', '23'),
('23678', 'SAN CARLOS', '23'),
('23686', 'SAN PELAYO', '23'),
('23807', 'TIERRALTA', '23'),
('23855', 'VALENCIA', '23');

```

```sql
INSERT INTO citiesormunicipalities (code, name, statereg_id) VALUES
('25001', 'AGUA DE DIOS', '25'),
('25019', 'ALBAN', '25'),
('25035', 'ANAPOIMA', '25'),
('25040', 'ANOLAIMA', '25'),
('25053', 'ARBELAEZ', '25'),
('25086', 'BELTRAN', '25'),
('25095', 'BITUIMA', '25'),
('25099', 'BOJACA', '25'),
('25120', 'CABRERA', '25'),
('25123', 'CACHIPAY', '25'),
('25126', 'CAJICA', '25'),
('25148', 'CAPARRAPI', '25'),
('25151', 'CAQUEZA', '25'),
('25154', 'CARMEN DE CARUPA', '25'),
('25168', 'CHAGUANI', '25'),
('25175', 'CHIA', '25'),
('25178', 'CHIPAQUE', '25'),
('25181', 'CHOACHI', '25'),
('25183', 'CHOCONTA', '25'),
('25200', 'COGUA', '25'),
('25214', 'COTA', '25'),
('25224', 'CUCUNUBA', '25'),
('25245', 'EL COLEGIO', '25'),
('25258', 'EL PEÑON', '25'),
('25260', 'EL ROSAL', '25'),
('25269', 'FACATATIVA', '25'),
('25279', 'FOMEQUE', '25'),
('25281', 'FOSCA', '25'),
('25286', 'FUNZA', '25'),
('25288', 'FUQUENE', '25'),
('25290', 'FUSAGASUGA', '25'),
('25293', 'GACHALA', '25'),
('25295', 'GACHANCIPA', '25'),
('25297', 'GACHETA', '25'),
('25299', 'GAMA', '25'),
('25307', 'GIRARDOT', '25'),
('25312', 'GRANADA', '25'),
('25317', 'GUACHETA', '25'),
('25320', 'GUADUAS', '25'),
('25322', 'GUASCA', '25'),
('25324', 'GUATAQUI', '25'),
('25326', 'GUATAVITA', '25'),
('25328', 'GUAYABAL DE SIQUIMA', '25'),
('25335', 'GUAYABETAL', '25'),
('25339', 'GUTIERREZ', '25'),
('25368', 'JERUSALEN', '25'),
('25372', 'JUNIN', '25'),
('25377', 'LA CALERA', '25'),
('25386', 'LA MESA', '25'),
('25394', 'LA PALMA', '25'),
('25398', 'LA PEÑA', '25'),
('25402', 'LA VEGA', '25'),
('25407', 'LENGUAZAQUE', '25'),
('25426', 'MACHETA', '25'),
('25430', 'MADRID', '25'),
('25436', 'MANTA', '25'),
('25438', 'MEDINA', '25'),
('25473', 'MOSQUERA', '25'),
('25483', 'NARIÑO', '25'),
('25486', 'NEMOCON', '25'),
('25488', 'NILO', '25'),
('25489', 'NIMAIMA', '25'),
('25491', 'NOCAIMA', '25'),
('25506', 'VENECIA', '25'),
('25513', 'PACHO', '25'),
('25518', 'PAIME', '25'),
('25524', 'PANDI', '25'),
('25530', 'PARATEBUENO', '25'),
('25535', 'PASCA', '25'),
('25572', 'PUERTO SALGAR', '25'),
('25580', 'PULI', '25'),
('25592', 'QUEBRADANEGRA', '25'),
('25594', 'QUETAME', '25'),
('25596', 'QUIPILE', '25'),
('25599', 'APULO', '25'),
('25612', 'RICAURTE', '25'),
('25645', 'SAN ANTONIO DEL TEQUENDAMA', '25'),
('25649', 'SAN BERNARDO', '25'),
('25653', 'SAN CAYETANO', '25'),
('25658', 'SAN FRANCISCO', '25'),
('25662', 'SAN JUAN DE RIO SECO', '25'),
('25718', 'SASAIMA', '25'),
('25736', 'SESQUILE', '25'),
('25740', 'SIBATE', '25'),
('25743', 'SILVANIA', '25'),
('25745', 'SIMIJACA', '25'),
('25754', 'SOACHA', '25'),
('25758', 'SOPO', '25'),
('25769', 'SUBACHOQUE', '25'),
('25772', 'SUESCA', '25'),
('25777', 'SUPATA', '25'),
('25779', 'SUSA', '25'),
('25781', 'SUTATAUSA', '25'),
('25785', 'TABIO', '25'),
('25793', 'TAUSA', '25'),
('25797', 'TENA', '25'),
('25799', 'TENJO', '25'),
('25805', 'TIBACUY', '25'),
('25807', 'TIBIRITA', '25'),
('25815', 'TOCAIMA', '25'),
('25817', 'TOCANCIPA', '25'),
('25823', 'TOPAIPI', '25'),
('25839', 'UBALA', '25'),
('25841', 'UBAQUE', '25'),
('25843', 'VILLA DE SAN DIEGO DE UBATE', '25'),
('25845', 'UNE', '25'),
('25851', 'UTICA', '25'),
('25862', 'VERGARA', '25'),
('25867', 'VIANI', '25'),
('25871', 'VILLAGOMEZ', '25'),
('25873', 'VILLAPINZON', '25'),
('25875', 'VILLETA', '25'),
('25878', 'VIOTA', '25'),
('25885', 'YACOPI', '25'),
('25898', 'ZIPACON', '25'),
('25899', 'ZIPAQUIRA', '25');

```

```sql
INSERT INTO citiesormunicipalities (code, name, statereg_id) VALUES
('27001', 'QUIBDO', '27'),
('27006', 'ACANDI', '27'),
('27025', 'ALTO BAUDO', '27'),
('27050', 'ATRATO', '27'),
('27073', 'BAGADO', '27'),
('27075', 'BAHIA SOLANO', '27'),
('27077', 'BAJO BAUDO', '27'),
('27086', 'BELEN DE BAJIRA', '27'),
('27099', 'BOJAYA', '27'),
('27135', 'EL CANTON DEL SAN PABLO', '27'),
('27150', 'CARMEN DEL DARIEN', '27'),
('27160', 'CERTEGUI', '27'),
('27205', 'CONDOTO', '27'),
('27245', 'EL CARMEN DE ATRATO', '27'),
('27250', 'EL LITORAL DEL SAN JUAN', '27'),
('27361', 'ISTMINA', '27'),
('27372', 'JURADO', '27'),
('27413', 'LLORO', '27'),
('27425', 'MEDIO ATRATO', '27'),
('27430', 'MEDIO BAUDO', '27'),
('27450', 'MEDIO SAN JUAN', '27'),
('27491', 'NOVITA', '27'),
('27495', 'NUQUI', '27'),
('27580', 'RIO IRO', '27'),
('27600', 'RIO QUITO', '27'),
('27615', 'RIOSUCIO', '27'),
('27660', 'SAN JOSE DEL PALMAR', '27'),
('27745', 'SIPI', '27'),
('27787', 'TADO', '27'),
('27800', 'UNGUIA', '27'),
('27810', 'UNION PANAMERICANA', '27');
```

```sql
INSERT INTO citiesormunicipalities (code, name, statereg_id) VALUES
('41 001', 'NEIVA', '41'),
('41 006', 'ACEVEDO', '41'),
('41 013', 'AGRADO', '41'),
('41 016', 'AIPE', '41'),
('41 020', 'ALGECIRAS', '41'),
('41 026', 'ALTAMIRA', '41'),
('41 078', 'BARAYA', '41'),
('41 132', 'CAMPOALEGRE', '41'),
('41 206', 'COLOMBIA', '41'),
('41 244', 'ELIAS', '41'),
('41 298', 'GARZON', '41'),
('41 306', 'GIGANTE', '41'),
('41 319', 'GUADALUPE', '41'),
('41 349', 'HOBO', '41'),
('41 357', 'IQUIRA', '41'),
('41 359', 'ISNOS', '41'),
('41 378', 'LA ARGENTINA', '41'),
('41 396', 'LA PLATA', '41'),
('41 483', 'NATAGA', '41'),
('41 503', 'OPORAPA', '41'),
('41 518', 'PAICOL', '41'),
('41 524', 'PALERMO', '41'),
('41 530', 'PALESTINA', '41'),
('41 548', 'PITAL', '41'),
('41 551', 'PITALITO', '41'),
('41 615', 'RIVERA', '41'),
('41 660', 'SALADOBLANCO', '41'),
('41 668', 'SAN AGUSTIN', '41'),
('41 676', 'SANTA MARIA', '41'),
('41 770', 'SUAZA', '41'),
('41 791', 'TARQUI', '41'),
('41 797', 'TESALIA', '41'),
('41 799', 'TELLO', '41'),
('41 801', 'TERUEL', '41'),
('41 807', 'TIMANA', '41'),
('41 872', 'VILLAVIEJA', '41'),
('41 885', 'YAGUARA', '41');

```

```sql
INSERT INTO citiesormunicipalities (code, name, statereg_id) VALUES
('44001', 'RIOHACHA', '44'),
('44035', 'ALBANIA', '44'),
('44078', 'BARRANCAS', '44'),
('44090', 'DIBULLA', '44'),
('44098', 'DISTRACCION', '44'),
('44110', 'EL MOLINO', '44'),
('44279', 'FONSECA', '44'),
('44378', 'HATONUEVO', '44'),
('44420', 'LA JAGUA DEL PILAR', '44'),
('44430', 'MAICAO', '44'),
('44560', 'MANAURE', '44'),
('44650', 'SAN JUAN DEL CESAR', '44'),
('44847', 'URIBIA', '44'),
('44855', 'URUMITA', '44'),
('44874', 'VILLANUEVA', '44');
```

```sql
INSERT INTO citiesormunicipalities (code, name, statereg_id) VALUES
('47001', 'SANTA MARTA', '47'),
('47030', 'ALGARROBO', '47'),
('47053', 'ARACATACA', '47'),
('47058', 'ARIGUANI', '47'),
('47161', 'CERRO SAN ANTONIO', '47'),
('47170', 'CHIVOLO', '47'),
('47189', 'CIENAGA', '47'),
('47205', 'CONCORDIA', '47'),
('47245', 'EL BANCO', '47'),
('47258', 'EL PIÑON', '47'),
('47268', 'EL RETEN', '47'),
('47288', 'FUNDACION', '47'),
('47318', 'GUAMAL', '47'),
('47460', 'NUEVA GRANADA', '47'),
('47541', 'PEDRAZA', '47'),
('47545', 'PIJIÑO DEL CARMEN', '47'),
('47551', 'PIVIJAY', '47'),
('47555', 'PLATO', '47'),
('47570', 'PUEBLOVIEJO', '47'),
('47605', 'REMOLINO', '47'),
('47660', 'SABANAS DE SAN ANGEL', '47'),
('47675', 'SALAMINA', '47'),
('47692', 'SAN SEBASTIAN', '47'),
('47703', 'SAN ZENON', '47'),
('47707', 'SANTA ANA', '47'),
('47720', 'SANTA BARBARA DE PINTO', '47'),
('47745', 'SITIONUEVO', '47'),
('47798', 'TENERIFE', '47'),
('47960', 'ZAPAYAN', '47'),
('47980', 'ZONA BANANERA', '47');

```

```sql
INSERT INTO citiesormunicipalities (code, name, statereg_id) VALUES
('50001', 'VILLAVICENCIO', '50'),
('50006', 'ACACIAS', '50'),
('50110', 'BARRANCA DE UPIA', '50'),
('50124', 'CABUYARO', '50'),
('50150', 'CASTILLA LA NUEVA', '50'),
('50223', 'CUBARRAL', '50'),
('50226', 'CUMARAL', '50'),
('50245', 'EL CALVARIO', '50'),
('50251', 'EL CASTILLO', '50'),
('50270', 'EL DORADO', '50'),
('50287', 'FUENTE DE ORO', '50'),
('50313', 'GRANADA', '50'),
('50318', 'GUAMAL', '50'),
('50325', 'MAPIRIPAN', '50'),
('50330', 'MESETAS', '50'),
('50350', 'LA MACARENA', '50'),
('50370', 'URIBE', '50'),
('50400', 'LEJANIAS', '50'),
('50450', 'PUERTO CONCORDIA', '50'),
('50568', 'PUERTO GAITAN', '50'),
('50573', 'PUERTO LOPEZ', '50'),
('50577', 'PUERTO LLERAS', '50'),
('50590', 'PUERTO RICO', '50'),
('50606', 'RESTREPO', '50'),
('50680', 'SAN CARLOS DE GUAROA', '50'),
('50683', 'SAN JUAN DE ARAMA', '50'),
('50686', 'SAN JUANITO', '50'),
('50689', 'SAN MARTIN', '50'),
('50711', 'VISTA HERMOSA', '50');


```

```sql
INSERT INTO citiesormunicipalities (code, name, statereg_id) VALUES
('52001', 'PASTO', '52'),
('52019', 'ALBAN', '52'),
('52022', 'ALDANA', '52'),
('52036', 'ANCUYA', '52'),
('52051', 'ARBOLEDA', '52'),
('52079', 'BARBACOAS', '52'),
('52083', 'BELEN', '52'),
('52110', 'BUESACO', '52'),
('52203', 'COLON', '52'),
('52207', 'CONSACA', '52'),
('52210', 'CONTADERO', '52'),
('52215', 'CORDOBA', '52'),
('52224', 'CUASPUD', '52'),
('52227', 'CUMBAL', '52'),
('52233', 'CUMBITARA', '52'),
('52240', 'CHACHAGÜI', '52'),
('52250', 'EL CHARCO', '52'),
('52254', 'EL PEÑOL', '52'),
('52256', 'EL ROSARIO', '52'),
('52258', 'EL TABLON DE GOMEZ', '52'),
('52260', 'EL TAMBO', '52'),
('52287', 'FUNES', '52'),
('52317', 'GUACHUCAL', '52'),
('52320', 'GUAITARILLA', '52'),
('52323', 'GUALMATAN', '52'),
('52352', 'ILES', '52'),
('52354', 'IMUES', '52'),
('52356', 'IPIALES', '52'),
('52378', 'LA CRUZ', '52'),
('52381', 'LA FLORIDA', '52'),
('52385', 'LA LLANADA', '52'),
('52390', 'LA TOLA', '52'),
('52399', 'LA UNION', '52'),
('52405', 'LEIVA', '52'),
('52411', 'LINARES', '52'),
('52418', 'LOS ANDES', '52'),
('52427', 'MAGÜI', '52'),
('52435', 'MALLAMA', '52'),
('52473', 'MOSQUERA', '52'),
('52480', 'NARIÑO', '52'),
('52490', 'OLAYA HERRERA', '52'),
('52506', 'OSPINA', '52'),
('52520', 'FRANCISCO PIZARRO', '52'),
('52540', 'POLICARPA', '52'),
('52560', 'POTOSÍ', '52'),
('52565', 'PROVIDENCIA', '52'),
('52573', 'PUERRES', '52'),
('52585', 'PUPIALES', '52'),
('52612', 'RICAURTE', '52'),
('52621', 'ROBERTO PAYAN', '52'),
('52678', 'SAMANIEGO', '52'),
('52683', 'SANDONA', '52'),
('52685', 'SAN BERNARDO', '52'),
('52687', 'SAN LORENZO', '52'),
('52693', 'SAN PABLO', '52'),
('52694', 'SAN PEDRO DE CARTAGO', '52'),
('52696', 'SANTA BARBARA', '52'),
('52699', 'SANTACRUZ', '52'),
('52720', 'SAPUYES', '52'),
('52786', 'TAMINANGO', '52'),
('52788', 'TANGUA', '52'),
('52835', 'TUMACO', '52'),
('52838', 'TUQUERRES', '52'),
('52885', 'YACUANQUER', '52');

```

```sql
INSERT INTO citiesormunicipalities (code, name, statereg_id) VALUES
('54001', 'CUCUTA', '54'),
('54003', 'ABREGO', '54'),
('54051', 'ARBOLEDAS', '54'),
('54099', 'BOCHALEMA', '54'),
('54109', 'BUCARASICA', '54'),
('54125', 'CACOTA', '54'),
('54128', 'CACHIRA', '54'),
('54172', 'CHINACOTA', '54'),
('54174', 'CHITAGA', '54'),
('54206', 'CONVENCION', '54'),
('54223', 'CUCUTILLA', '54'),
('54239', 'DURANIA', '54'),
('54245', 'EL CARMEN', '54'),
('54250', 'EL TARRA', '54'),
('54261', 'EL ZULIA', '54'),
('54313', 'GRAMALOTE', '54'),
('54344', 'HACARI', '54'),
('54347', 'HERRAN', '54'),
('54377', 'LABATECA', '54'),
('54385', 'LA ESPERANZA', '54'),
('54398', 'LA PLAYA', '54'),
('54405', 'LOS PATIOS', '54'),
('54418', 'LOURDES', '54'),
('54480', 'MUTISCUA', '54'),
('54498', 'OCAÑA', '54'),
('54518', 'PAMPLONA', '54'),
('54520', 'PAMPLONITA', '54'),
('54553', 'PUERTO SANTANDER', '54'),
('54599', 'RAGONVALIA', '54'),
('54660', 'SALAZAR', '54'),
('54670', 'SAN CALIXTO', '54'),
('54673', 'SAN CAYETANO', '54'),
('54680', 'SANTIAGO', '54'),
('54720', 'SARDINATA', '54'),
('54743', 'SILOS', '54'),
('54800', 'TEORAMA', '54'),
('54810', 'TIBU', '54'),
('54820', 'TOLEDO', '54'),
('54871', 'VILLA CARO', '54'),
('54874', 'VILLA DEL ROSARIO', '54');

```

```sql
INSERT INTO citiesormunicipalities (code, name, statereg_id) VALUES
('86001', 'MOCOA', '86'),
('86219', 'COLON', '86'),
('86320', 'ORITO', '86'),
('86568', 'PUERTO ASIS', '86'),
('86569', 'PUERTO CAICEDO', '86'),
('86571', 'PUERTO GUZMAN', '86'),
('86573', 'LEGUIZAMO', '86'),
('86749', 'SIBUNDOY', '86'),
('86755', 'SAN FRANCISCO', '86'),
('86757', 'SAN MIGUEL', '86'),
('86760', 'SANTIAGO', '86'),
('86865', 'VALLE DEL GUAMUEZ', '86'),
('86885', 'VILLAGARZON', '86');


```

```sql
INSERT INTO citiesormunicipalities (code, name, statereg_id) VALUES
('88001', 'SAN ANDRES', '88'),
('88564', 'PROVIDENCIA', '88');


```

```sql
INSERT INTO citiesormunicipalities (code, name, statereg_id) VALUES
('91001', 'LETICIA', '91'),
('91263', 'EL ENCANTO', '91'),
('91405', 'LA CHORRERA', '91'),
('91407', 'LA PEDRERA', '91'),
('91430', 'LA VICTORIA', '91'),
('91460', 'MIRITI - PARANA', '91'),
('91530', 'PUERTO ALEGRIA', '91'),
('91536', 'PUERTO ARICA', '91'),
('91540', 'PUERTO NARIÑO', '91'),
('91669', 'PUERTO SANTANDER', '91'),
('91798', 'TARAPACA', '91');

```

```sql
INSERT INTO citiesormunicipalities (code, name, statereg_id) VALUES
('94001', 'INIRIDA', '94'),
('94343', 'BARRANCO MINAS', '94'),
('94663', 'MAPIRIPANA', '94'),
('94883', 'SAN FELIPE', '94'),
('94884', 'PUERTO COLOMBIA', '94'),
('94885', 'LA GUADALUPE', '94'),
('94886', 'CACAHUAL', '94'),
('94887', 'PANA PANA', '94'),
('94888', 'MORICHAL', '94');

```

```sql
INSERT INTO citiesormunicipalities (code, name, statereg_id) VALUES
('95001', 'SAN JOSE DEL GUAVIARE', '95'),
('95015', 'CALAMAR', '95'),
('95025', 'EL RETORNO', '95'),
('95200', 'MIRAFLORES', '95');

```

```sql
INSERT INTO citiesormunicipalities (code, name, statereg_id) VALUES
('97001', 'MITU', '97'),
('97161', 'CARURU', '97'),
('97511', 'PACOA', '97'),
('97666', 'TARAIRA', '97'),
('97777', 'PAPUNAUA', '97'),
('97889', 'YAVARATE', '97');

```

```sql
INSERT INTO citiesormunicipalities (code, name, statereg_id) VALUES
('99001', 'PUERTO CARREÑO', '99'),
('99524', 'LA PRIMAVERA', '99'),
('99624', 'SANTA ROSALIA', '99'),
('99773', 'CUMARIBO', '99');

```



##### **ESTADOS UNIDOS** 

```sql
INSERT INTO stateregions (code, name, country_id, code_3166_2, subdivision_id) VALUES
('AL', 'Alabama', 840, 'US-AL', 3),
('AK', 'Alaska', 840, 'US-AK', 3),
('AS', 'American Samoa', 840, 'US-AS', 4),
('AZ', 'Arizona', 840, 'US-AZ', 3),
('AR', 'Arkansas', 840, 'US-AR', 3),
('CA', 'California', 840, 'US-CA', 3),
('CO', 'Colorado', 840, 'US-CO', 3),
('CT', 'Connecticut', 840, 'US-CT', 3),
('DE', 'Delaware', 840, 'US-DE', 3),
('DC', 'District of Columbia', 840, 'US-DC', 4),
('FL', 'Florida', 840, 'US-FL', 3),
('GA', 'Georgia', 840, 'US-GA', 3),
('GU', 'Guam', 840, 'US-GU', 4),
('HI', 'Hawaii', 840, 'US-HI', 3),
('ID', 'Idaho', 840, 'US-ID', 3),
('IL', 'Illinois', 840, 'US-IL', 3),
('IN', 'Indiana', 840, 'US-IN', 3),
('IA', 'Iowa', 840, 'US-IA', 3),
('KS', 'Kansas', 840, 'US-KS', 3),
('KY', 'Kentucky', 840, 'US-KY', 3),
('LA', 'Louisiana', 840, 'US-LA', 3),
('ME', 'Maine', 840, 'US-ME', 3),
('MD', 'Maryland', 840, 'US-MD', 3),
('MA', 'Massachusetts', 840, 'US-MA', 3),
('MI', 'Michigan', 840, 'US-MI', 3),
('MN', 'Minnesota', 840, 'US-MN', 3),
('MS', 'Mississippi', 840, 'US-MS', 3),
('MO', 'Missouri', 840, 'US-MO', 3),
('MT', 'Montana', 840, 'US-MT', 3),
('NE', 'Nebraska', 840, 'US-NE', 3),
('NV', 'Nevada', 840, 'US-NV', 3),
('NH', 'New Hampshire', 840, 'US-NH', 3),
('NJ', 'New Jersey', 840, 'US-NJ', 3),
('NM', 'New Mexico', 840, 'US-NM', 3),
('NY', 'New York', 840, 'US-NY', 3),
('NC', 'North Carolina', 840, 'US-NC', 3),
('ND', 'North Dakota', 840, 'US-ND', 3),
('MP', 'Northern Mariana Islands', 840, 'US-MP', 4),
('OH', 'Ohio', 840, 'US-OH', 3),
('OK', 'Oklahoma', 840, 'US-OK', 3),
('OR', 'Oregon', 840, 'US-OR', 3),
('PA', 'Pennsylvania', 840, 'US-PA', 3),
('PR', 'Puerto Rico', 840, 'US-PR', 4),
('RI', 'Rhode Island', 840, 'US-RI', 3),
('SC', 'South Carolina', 840, 'US-SC', 3),
('SD', 'South Dakota', 840, 'US-SD', 3),
('TN', 'Tennessee', 840, 'US-TN', 3),
('TX', 'Texas', 840, 'US-TX', 3),
('UM', 'United States Minor Outlying Islands', 840, 'US-UM', 4),
('UT', 'Utah', 840, 'US-UT', 3),
('VT', 'Vermont', 840, 'US-VT', 3),
('VI', 'Virgin Islands, U.S.', 840, 'US-VI', 4),
('VA', 'Virginia', 840, 'US-VA', 3),
('WA', 'Washington', 840, 'US-WA', 3),
('WV', 'West Virginia', 840, 'US-WV', 3),
('WI', 'Wisconsin', 840, 'US-WI', 3),
('WY', 'Wyoming', 840, 'US-WY', 3);

```

```sql
INSERT INTO citiesormunicipalities (code, name, statereg_id) VALUES
('NY001', 'New York', 'NY'),
('CA002', 'Los Angeles', 'CA'),
('IL003', 'Chicago', 'IL'),
('TX004', 'Houston', 'TX'),
('AZ005', 'Phoenix', 'AZ'),
('PA006', 'Philadelphia', 'PA'),
('TX007', 'San Antonio', 'TX'),
('CA008', 'San Diego', 'CA'),
('TX009', 'Dallas', 'TX'),
('CA010', 'San Jose', 'CA');

```

```sql
INSERT INTO addresses (street, city_id, postal_code) VALUES
('1234 W Camelback Rd', 'AZ005', '85017'),
('2400 E Van Buren St', 'AZ005', '85008'),

('1100 S Broadway', 'CA002', '90015'),
('8800 Sunset Blvd', 'CA002', '90069'),

('5055 Governor Dr', 'CA008', '92122'),
('1075 Camino del Rio S', 'CA008', '92108'),

('1510 N First St', 'CA010', '95112'),
('3000 S Bascom Ave', 'CA010', '95008'),

('233 S Wacker Dr', 'IL003', '60606'),
('1050 W Belmont Ave', 'IL003', '60657'),

('1600 Market St', 'PA006', '19103'),
('2800 Island Ave', 'PA006', '19153'),

('3333 Richmond Ave', 'TX004', '77098'),
('945 McKinney St', 'TX004', '77002'),

('200 E Market St', 'TX007', '78205'),
('12345 Nacogdoches Rd', 'TX007', '78217'),

('500 Main St', 'TX009', '75201'),
('1800 N Greenville Ave', 'TX009', '75081');

```

```sql
INSERT INTO audiences (description) VALUES
('Músicos principiantes'),
('Músicos profesionales'),
('Estudiantes de música'),
('Padres que compran instrumentos para sus hijos'),
('Profesores de música'),
('Amantes de la música clásica'),
('Amantes del rock'),
('Productores de música electrónica'),
('Iglesias y grupos religiosos'),
('Escuelas y academias de música'),
('Bandas y orquestas'),
('DJ y productores de estudio'),
('Compradores de guitarras eléctricas'),
('Compradores de pianos digitales'),
('Compradores de instrumentos de viento');
```

```sql
INSERT INTO customers (id, name, city_id, audience_id) VALUES
(1, 'Emily Johnson', '05002', 13),
(2, 'Jessica Smith', '25040', 1),
(3, 'Jessica Romero', '05004', 15),
(4, 'Laura Gómez', '25040', 13),
(5, 'Sarah Brown', '05030', 1),
(6, 'Robert Smith', '25053', 5),
(7, 'Luis Smith', '05001', 12),
(8, 'Michael Romero', '05004', 12),
(9, 'Luis Torres', '25053', 2),
(10, 'Juan Taylor', '25040', 14),
(11, 'Ana Romero', '05030', 8),
(12, 'James Gómez', '05021', 7),
(13, 'Emily Taylor', '05002', 10),
(14, 'Jessica Gómez', '05002', 8),
(15, 'Michael Pérez', '25035', 6),
(16, 'Marta Romero', '25040', 6),
(17, 'Jessica Romero', '25035', 6),
(18, 'Emily Martínez', '25035', 15),
(19, 'Jessica Pérez', '25035', 2),
(20, 'Laura Taylor', '05001', 7),
(21, 'Marta Wilson', '05002', 9),
(22, 'Sarah Romero', '25053', 14),
(23, 'Marta Torres', '05021', 15),
(24, 'James Gómez', '05021', 2),
(25, 'Carlos Davis', '05004', 10),
(26, 'Marta Torres', '25001', 7),
(27, 'Sarah Brown', '05001', 13),
(28, 'Ana Wilson', '05030', 7),
(29, 'Marta Brown', '05004', 7),
(30, 'Marta Brown', '25001', 4),
(31, 'Luis Wilson', 'CA008', 14),
(32, 'James Martínez', 'TX009', 2),
(33, 'Michael Torres', 'TX009', 2),
(34, 'Marta Martínez', 'CA010', 6),
(35, 'Luis Smith', 'TX009', 4),
(36, 'Emily Pérez', 'CA010', 6),
(37, 'Emily Romero', 'PA006', 15),
(38, 'Laura Martínez', 'CA008', 10),
(39, 'Carlos Romero', 'TX004', 15),
(40, 'Ana Smith', 'CA008', 13),
(41, 'Sarah Davis', 'PA006', 2),
(42, 'Laura Davis', 'CA008', 3),
(43, 'Ana Pérez', 'IL003', 15),
(44, 'Robert Davis', 'IL003', 9),
(45, 'Marta Gómez', 'CA010', 3),
(46, 'Juan Torres', 'PA006', 12),
(47, 'Laura Johnson', 'TX004', 6),
(48, 'Juan Wilson', 'TX004', 1),
(49, 'Luis Torres', 'IL003', 7),
(50, 'Robert Brown', 'CA010', 13),
(51, 'Marta Brown', 'TX004', 2),
(52, 'Marta Pérez', 'IL003', 14),
(53, 'Carlos Martínez', 'IL003', 14),
(54, 'Marta Brown', 'TX007', 6),
(55, 'Michael Romero', 'PA006', 13),
(56, 'Sarah Torres', 'TX004', 14),
(57, 'James Martínez', 'TX007', 12),
(58, 'Ana Davis', 'AZ005', 11),
(59, 'Luis Johnson', 'AZ005', 3),
(60, 'Jessica Wilson', 'AZ005', 9);

```

```sql
INSERT INTO addresses (id, street, city_id, postal_code) VALUES
(1, 'Calle 10 #5-20', '05001', '050001'),
(2, 'Carrera 45 #30-22', '05002', '050002'),
(3, 'Diagonal 7 #12-80', '05004', '050004'),
(4, 'Transversal 3 #18-11', '05021', '050021'),
(5, 'Calle 20 #8-16', '05030', '050030'),
(6, 'Carrera 12 #7-40', '05031', '050031'),
(7, 'Calle 15 #6-33', '05034', '050034'),
(8, 'Carrera 9 #4-17', '05036', '050036'),
(9, 'Calle 11 #2-28', '05038', '050038'),
(10, 'Carrera 7 #5-10', '05040', '050040'),
(11, 'Calle 21 #6-18', '05042', '050042'),
(12, 'Carrera 3 #9-05', '05044', '050044'),
(13, 'Calle 13 #2-14', '05045', '050045'),
(14, 'Carrera 11 #10-30', '05051', '050051'),
(15, 'Transversal 6 #3-22', '05055', '050055'),
(16, 'Calle 17 #8-60', '05059', '050059'),
(17, 'Carrera 8 #4-19', '05079', '050079'),
(18, 'Calle 25 #7-08', '05086', '050086'),
(19, 'Carrera 6 #5-42', '05088', '050088'),
(20, 'Calle 12 #3-70', '05091', '050091'),
(21, 'Carrera 13 #2-33', '05093', '050093'),
(22, 'Calle 9 #7-90', '05101', '051001'),
(23, 'Carrera 15 #10-22', '05107', '051007'),
(24, 'Calle 18 #6-40', '05113', '051013'),
(25, 'Carrera 10 #11-08', '05120', '051020'),
(26, '123 W Jefferson St', 'AZ005', '85003'),     
(27, '456 S Figueroa St', 'CA002', '90017'),       
(28, '789 Broadway', 'CA008', '92101'),            
(29, '101 N Market St', 'CA010', '95113'),        
(30, '202 W Madison St', 'IL003', '60606'),         
(31, '350 5th Ave', 'NY001', '10001'),              
(32, '1500 Market St', 'PA006', '19102'),           
(33, '910 Louisiana St', 'TX004', '77002'),         
(34, '100 Military Plaza', 'TX007', '78205'),       
(35, '300 Main St', 'TX009', '75201'), 
(36, '200 W Washington St', 'AZ005', '85003'),       
(37, '640 S Sunset Blvd', 'CA002', '90028'),        
(38, '500 Harbor Dr', 'CA008', '92101'),             
(39, '75 E Santa Clara St', 'CA010', '95113'),       
(40, '233 S Wacker Dr', 'IL003', '60606'),           
(41, '11 Wall Street', 'NY001', '10005'),            
(42, '1701 John F Kennedy Blvd', 'PA006', '19103'),  
(43, '401 Franklin St', 'TX004', '77002'),          
(44, '303 Alamo Plaza', 'TX007', '78205'),           
(45, '2200 Ross Ave', 'TX009', '75201'),
(46, '100 N 3rd St', 'AZ005', '85004'),              
(47, '6801 Hollywood Blvd', 'CA002', '90028'),       
(48, '1100 Coast Blvd', 'CA008', '92037'),            
(49, '150 W San Carlos St', 'CA010', '95113'),        
(50, '1400 S Lake Shore Dr', 'IL003', '60605'),       
(51, '405 Lexington Ave', 'NY001', '10174'),          
(52, '3501 Boardwalk', 'PA006', '19148'),            
(53, '1500 McKinney St', 'TX004', '77010'),       
(54, '849 E Commerce St', 'TX007', '78205'),         
(55, '411 Elm St', 'TX009', '75202'),                 
(56, '1 E Washington St', 'AZ005', '85004'),          
(57, '333 S Spring St', 'CA002', '90013'),        
(58, '1350 El Prado', 'CA008', '92101'),              
(59, '1700 Alum Rock Ave', 'CA010', '95116'),         
(60, '1200 S Lake Shore Dr', 'IL003', '60605');       
```

###### customer_phones

```sql
INSERT INTO customer_phones (customer_id, phone) VALUES
(1, '+57-4-321-4567'),
(2, '+57-1-322-7890'),
(3, '+57-4-300-2345'),
(4, '+57-1-320-9988'),
(5, '+57-4-322-1122'),
(6, '+57-1-300-4455'),
(7, '+57-4-313-7890'),
(8, '+57-4-320-1111'),
(9, '+57-1-301-2222'),
(10, '+57-1-302-3333'),
(11, '+57-4-304-4444'),
(12, '+57-4-305-5555'),
(13, '+57-4-306-6666'),
(14, '+57-4-307-7777'),
(15, '+57-1-310-8888'),
(16, '+57-1-311-9999'),
(17, '+57-1-312-0000'),
(18, '+57-2-313-1010'),
(19, '+57-4-314-2020'),
(20, '+57-4-315-3030'),
(21, '+57-4-316-4040'),
(22, '+57-1-317-5050'),
(23, '+57-4-318-6060'),
(24, '+57-4-319-7070'),
(25, '+57-4-320-8080'),
(26, '+57-5-321-9090'),
(27, '+57-4-322-0101'),
(28, '+57-4-323-1212'),
(29, '+57-4-324-2323'),
(30, '+57-5-325-3434'),
(31, '+1-619-555-0101'),
(32, '+1-214-555-0202'),
(33, '+1-214-555-0303'),
(34, '+1-408-555-0404'),
(35, '+1-214-555-0505'),
(36, '+1-408-555-0606'),
(37, '+1-215-555-0707'),
(38, '+1-619-555-0808'),
(39, '+1-713-555-0909'),
(40, '+1-619-555-1010'),
(41, '+1-215-555-1111'),
(42, '+1-619-555-1212'),
(43, '+1-312-555-1313'),
(44, '+1-312-555-1414'),
(45, '+1-408-555-1515'),
(46, '+1-215-555-1616'),
(47, '+1-713-555-1717'),
(48, '+1-713-555-1818'),
(49, '+1-312-555-1919'),
(50, '+1-408-555-2020'),
(51, '+1-713-555-2121'),
(52, '+1-312-555-2222'),
(53, '+1-312-555-2323'),
(54, '+1-210-555-2424'),
(55, '+1-215-555-2525'),
(56, '+1-713-555-2626'),
(57, '+1-210-555-2727'),
(58, '+1-602-555-2828'),
(59, '+1-602-555-2929'),
(60, '+1-602-555-3030');


```

###### customer_emails

```sql
INSERT INTO customer_emails (customer_id, email) VALUES
(1, 'emily.johnson@cliente.com'),
(2, 'jessica.smith@cliente.com'),
(3, 'jessica.romero@cliente.com'),
(4, 'laura.gomez@cliente.com'),
(5, 'sarah.brown@cliente.com'),
(6, 'robert.smith@cliente.com'),
(7, 'luis.smith@cliente.com'),
(8, 'michael.romero@cliente.com'),
(9, 'luis.torres@cliente.com'),
(10, 'juan.taylor@cliente.com'),
(11, 'ana.romero@cliente.com'),
(12, 'james.gomez@cliente.com'),
(13, 'emily.taylor@cliente.com'),
(14, 'jessica.gomez@cliente.com'),
(15, 'michael.perez@cliente.com'),
(16, 'marta.romero@cliente.com'),
(17, 'jessica.romero2@cliente.com'),
(18, 'emily.martinez@cliente.com'),
(19, 'jessica.perez@cliente.com'),
(20, 'laura.taylor@cliente.com'),
(21, 'marta.wilson@cliente.com'),
(22, 'sarah.romero@cliente.com'),
(23, 'marta.torres@cliente.com'),
(24, 'james.gomez2@cliente.com'),
(25, 'carlos.davis@cliente.com'),
(26, 'marta.torres2@cliente.com'),
(27, 'sarah.brown2@cliente.com'),
(28, 'ana.wilson@cliente.com'),
(29, 'marta.brown@cliente.com'),
(30, 'marta.brown2@cliente.com'),
(31, 'luis.wilson@cliente.com'),
(32, 'james.martinez@cliente.com'),
(33, 'michael.torres@cliente.com'),
(34, 'marta.martinez@cliente.com'),
(35, 'luis.smith2@cliente.com'),
(36, 'emily.perez@cliente.com'),
(37, 'emily.romero@cliente.com'),
(38, 'laura.martinez@cliente.com'),
(39, 'carlos.romero@cliente.com'),
(40, 'ana.smith@cliente.com'),
(41, 'sarah.davis@cliente.com'),
(42, 'laura.davis@cliente.com'),
(43, 'ana.perez@cliente.com'),
(44, 'robert.davis@cliente.com'),
(45, 'marta.gomez@cliente.com'),
(46, 'juan.torres@cliente.com'),
(47, 'laura.johnson@cliente.com'),
(48, 'juan.wilson@cliente.com'),
(49, 'luis.torres2@cliente.com'),
(50, 'robert.brown@cliente.com'),
(51, 'marta.brown3@cliente.com'),
(52, 'marta.perez@cliente.com'),
(53, 'carlos.martinez@cliente.com'),
(54, 'marta.brown4@cliente.com'),
(55, 'michael.romero2@cliente.com'),
(56, 'sarah.torres@cliente.com'),
(57, 'james.martinez2@cliente.com'),
(58, 'ana.davis@cliente.com'),
(59, 'luis.johnson@cliente.com'),
(60, 'jessica.wilson@cliente.com');

```

###### memberships

```sqlite
INSERT INTO memberships (id, name, description) VALUES 
(1, 'Basic', 'Standard access with limited features and support.'),
(2, 'Premium', 'Full access to all features with priority support.'),
(3, 'Gold', 'Premium access plus exclusive content and early event access.');
```

benefits

```sql
INSERT INTO benefits (id, description, detail) VALUES
-- Para membresía Basic
(1, 'Standard Access', 'Access to basic platform features suitable for casual users.'),
(2, 'Email Support', 'Receive support via email with a 48-hour response time.'),
(3, 'Monthly Newsletter', 'Stay updated with a monthly email newsletter containing news and tips.'),

-- Para membresía Premium
(4, 'Full Access', 'Unlock all platform features with no restrictions.'),
(5, 'Priority Support', 'Get responses within 24 hours via email or live chat.'),
(6, 'Premium Content', 'Access to exclusive tutorials, webinars, and digital downloads.'),

-- Para membresía Gold
(7, 'Early Access', 'Be the first to try new features and tools before public release.'),
(8, 'Exclusive Events', 'Invitation to member-only events, workshops, and webinars.'),
(9, 'Loyalty Rewards', 'Earn points and gifts for continued membership and activity.');

```

###### unitofmesaure

```sql
INSERT INTO unitofmeasure (id, description) VALUES
(1, 'Unidad'),
(2, 'Par'),
(3, 'Paquete'),
(4, 'Metro'),
(5, 'Juego'),
(6, 'Caja'),
(7, 'Rollo');

```

###### categories

```sql
INSERT INTO categories (id, description) VALUES
(1, 'Cuerdas'),
(2, 'Percusión'),
(3, 'Viento'),
(4, 'Teclados'),
(5, 'Accesorios'),
(6, 'Audio y cables'),
(7, 'Estuches y fundas'),
(8, 'Micrófonos'),
(9, 'Equipos de grabación'),
(10, 'Iluminación');

```

###### typeidentifications

```sql
INSERT INTO typeidentifications (id, description, suffix) VALUES
(1, 'Número de Identificación Tributaria', 'NIT'),
(2, 'Cédula de Ciudadanía', 'CC'),
(3, 'Cédula de Extranjería', 'CE'),
(4, 'Pasaporte', 'PA'),
(5, 'Registro Único Tributario', 'RUT');

```

###### products



```sql
INSERT INTO products (id, name, detail, price, price_usd, category_id, image) VALUES
(1, 'Guitarra Acústica Yamaha F310', 'Guitarra acústica de cuerpo completo con cuerdas de acero, ideal para principiantes y estudiantes.', 750000, 187.50, 1, 'https://www.sweetwater.com/images/items/F310INT/1.jpg'),
(2, 'Batería Acústica Pearl Roadshow 5‑pz', 'Set completo de batería acústica 5 piezas con platillos incluidos, ideal para estudios o presentaciones en vivo.', 3200000, 800.00, 2, 'https://www.pearldrum.com/global/products/drum-sets/roadshow/roadshow.jpg'),
(3, 'Saxofón Alto Yamaha YAS‑280', 'Saxofón alto de excelente calidad para estudiantes avanzados, con estuche rígido incluido.', 5100000, 1275.00, 3, 'https://cdn.shopify.com/s/files/1/0025/9950/7296/products/YAS280_1024x.jpg'),
(4, 'Teclado Casio CT‑X700', 'Teclado portátil de 61 teclas con polifonía de 48 notas, 600 tonos y 195 ritmos.', 950000, 237.50, 4, 'https://static.casio.com/res/2020/745/CTX700K.jpg'),
(5, 'Púas Fender Heavy (x12)', 'Paquete de 12 púas Fender de celuloide con grosor heavy, ideales para guitarristas de rock.', 35000, 8.75, 5, 'https://cdn.shoplightspeed.com/shops/607265/files/33198547/fender-351-shape-media-12-pack.jpg'),
(6, 'Cable Instrumento Planet Waves 3 m', 'Cable profesional para guitarra o bajo de 1/4", longitud 3 metros, conectores niquelados.', 68000, 17.00, 6, 'https://www.planetwaves.com/media/catalog/product/cache/540x405/dci5wequfzpqvb6nhlid/p/w/pws_24003_1.jpg'),
(7, 'Estuche Rígido para Guitarra Clásica', 'Estuche protector tipo maleta para guitarra clásica, con interior acolchado y cerradura.', 210000, 52.50, 7, 'https://images.musicstore.de/0001689/rigid-classical-guitar-case.jpg'),
(8, 'Micrófono Shure SM58', 'Micrófono dinámico cardioide legendario para voz, resistente y confiable.', 520000, 130.00, 8, 'https://www.shure.com/-/media/project/shure/products/microphones/sm/sm58/gallery/sm58-gallery-black-variant.png'),
(9, 'Interfaz Focusrite Scarlett 2i2 Gen3', 'Interfaz de audio USB con dos entradas XLR/Jack y salidas balanceadas, ideal para home studio.', 870000, 217.50, 9, 'https://cdn.focusrite.com/archived/scarlett-2i2-3rd-gen.png'),
(10, 'Luz LED Par Chauvet DJ SlimPAR 56', 'Luz LED compacta de escenario con mezcla de colores RGB, ideal para eventos en vivo.', 390000, 97.50, 10, 'https://www.chauvetdj.com/media/catalog/product/cache/51c2d6f5b59558ca85f7b7b2e795efee/s/l/slimpar56_01.jpg')(11, 'Guitarra Eléctrica Fender Stratocaster Player', 'Cuerpo de aliso, mástil de arce, ideal para estilos versátiles.', 2850000, 712.50, 1, 'https://www.fmicassets.com/Damroot/LgJpg/10001/0144502513_fen_ins_frt_1_rr.jpg'),
(12, 'Batería Acústica Mapex Tornado', 'Set completo 5 piezas con hardware, ideal para principiantes.', 1950000, 487.50, 2, 'https://mapexdrums.com/media/catalog/product/cache/329c37773e6f7a3e5429a6b82b25e2f5/t/o/tornado_22-1.jpg'),
(13, 'Clarinete Yamaha YCL-255', 'Clarinete en Si bemol ideal para estudiantes, sonido cálido y fácil ejecución.', 2250000, 562.50, 3, 'https://cdn.shopify.com/s/files/1/0286/9345/5297/products/YCL255_1024x1024.jpg'),
(14, 'Yamaha PSR-E373', 'Teclado portátil con 622 voces, funciones educativas y conectividad USB MIDI.', 1120000, 280.00, 4, 'https://usa.yamaha.com/files/PSR-E373_front_1ca62bb93021f8dcf3c06be8409e1f59.jpg'),
(15, 'Capo Dunlop Trigger Curvo', 'Cejuela para guitarra acústica o eléctrica, diseño rápido de abrazadera.', 85000, 21.25, 5, 'https://cdn.shopify.com/s/files/1/0502/6442/7595/products/triggercapo_1024x1024.jpg'),
(16, 'Cable Mogami Gold 3m', 'Cable premium para guitarra/bajo con conectores de oro.', 150000, 37.50, 6, 'https://cdn.shopify.com/s/files/1/0267/9807/1627/products/Mogami-Gold-Instrument-10_1024x1024.jpg'),
(17, 'Estuche Gator ABS para guitarra eléctrica', 'Estuche rígido ABS con interior moldeado y cerraduras metálicas.', 310000, 77.50, 7, 'https://images.musicstore.de/images/1280/gator-gc-electric-case-fuer-e-gitarre_1.jpg'),
(18, 'Micrófono dinámico Sennheiser e835', 'Micrófono vocal cardioide con alta ganancia antes del feedback.', 470000, 117.50, 8, 'https://assets.sennheiser.com/img/6010/product_detail_x1_desktop_sqr/e835s_square_front.jpg'),
(19, 'Interfaz Behringer UMC22', 'Interface USB con preamplificador MIDAS y entrada combo XLR/Jack.', 240000, 60.00, 9, 'https://media.music-group.com/media/PLM/data/images/products/P0AUX/UMC22_P0AUX_Top_L.jpg'),
(20, 'Mini Par LED RGB U´King ZQ-B54', 'Luz LED para escenario con efectos automáticos y control DMX.', 160000, 40.00, 10, 'https://m.media-amazon.com/images/I/61UnabQ9ojL._AC_SL1500_.jpg');

```

###### categories_polls

```sql
INSERT INTO categories_polls (id, name) VALUES
(1, 'Satisfacción del cliente'),
(2, 'Calidad del producto'),
(3, 'Atención al cliente'),
(4, 'Tiempo de entrega'),
(5, 'Precio vs. calidad');

```

###### periods

```sql
INSERT INTO periods (id, name) VALUES
(1, '15 días'),
(2, '30 días'),
(3, '3 meses'),
(4, '6 meses'),
(5, '12 meses');

```

###### membershipperiods

"Los precios están en moneda colombiana"

```sql
INSERT INTO membershipperiods (membership_id, period_id, price) VALUES
-- Membresía Básica
(1, 1, 10000),
(1, 2, 18000),
(1, 3, 50000),
(1, 4, 90000),
(1, 5, 160000),

-- Membresía Premium
(2, 1, 15000),
(2, 2, 28000),
(2, 3, 75000),
(2, 4, 135000),
(2, 5, 240000),

-- Membresía Pro
(3, 1, 20000),
(3, 2, 35000),
(3, 3, 95000),
(3, 4, 180000),
(3, 5, 320000);

```

audiencebenefits

```sql
INSERT INTO audiencebenefits (audience_id, benefit_id) VALUES
-- 1. Músicos principiantes
(1, 1), -- Standard Access
(1, 2), -- Email Support
(1, 3), -- Monthly Newsletter

-- 2. Músicos profesionales
(2, 4), -- Full Access
(2, 5), -- Priority Support
(2, 6), -- Premium Content

-- 3. Estudiantes de música
(3, 1),
(3, 3),
(3, 6),

-- 4. Padres que compran instrumentos para sus hijos
(4, 2),
(4, 3),

-- 5. Profesores de música
(5, 4),
(5, 6),
(5, 8),

-- 6. Amantes de la música clásica
(6, 1),
(6, 3),
(6, 9),

-- 7. Amantes del rock
(7, 1),
(7, 3),

-- 8. Productores de música electrónica
(8, 4),
(8, 6),
(8, 7),

-- 9. Iglesias y grupos religiosos
(9, 2),
(9, 3),

-- 10. Escuelas y academias de música
(10, 4),
(10, 5),
(10, 8),

-- 11. Bandas y orquestas
(11, 1),
(11, 4),

-- 12. DJ y productores de estudio
(12, 4),
(12, 6),
(12, 7),

-- 13. Compradores de guitarras eléctricas
(13, 6),
(13, 7),

-- 14. Compradores de pianos digitales
(14, 3),
(14, 6),

-- 15. Compradores de instrumentos de viento
(15, 1),
(15, 3);

```

###### polls

```sql
INSERT INTO polls (id, name, description, isactive, categorypoll_id) VALUES
(1, 'Instrumentos Favoritos', 'Encuesta para conocer los instrumentos musicales más populares entre nuestros usuarios.', TRUE, 1),
(2, 'Satisfacción de Membresías', 'Cuéntanos cómo ha sido tu experiencia con nuestras membresías.', TRUE, 2),
(3, 'Nuevos Productos Deseados', 'Queremos saber qué productos te gustaría ver en nuestra tienda.', TRUE, 3),
(4, 'Calidad del Servicio', 'Evalúa la atención al cliente, envíos y experiencia de compra.', FALSE, 2),
(5, 'Eventos Musicales', '¿Qué tipo de eventos musicales te gustaría que organicemos o apoyemos?', TRUE, 4),
(6, 'Calidad del Producto', 'Evalúa la calidad de los productos adquiridos en la tienda.', TRUE, 2);

```

###### membershipbenefits

```sql
INSERT INTO membershipbenefits (membership_id, period_id, audience_id, benefit_id) VALUES

(1, 1, 3, 1),  
(1, 1, 3, 3),  

(2, 3, 2, 1),  
(2, 3, 2, 2),  
(2, 3, 2, 5),  

(3, 4, 10, 4), 
(3, 4, 10, 6), 
(3, 4, 10, 8), 


(3, 2, 12, 4), 
(3, 2, 12, 7); 

```

###### companies

```sql
INSERT INTO companies (id, name, type_id, category_id, city_id, audience_id) VALUES
(1, 'Melodías del Norte S.A.S.', 1, 2, '05001', 10),  
(2, 'Ritmo Total Ltda.', 2, 3, '25040', 2),          
(3, 'Harmony Records Inc.', 3, 1, 'TX009', 8),        
(4, 'Sonarte Colombia', 1, 5, '05004', 4),           
(5, 'DJ Factory Group', 2, 4, 'PA006', 12);          

```

###### quality_products

```sql
INSERT INTO quality_products (product_id, customer_id, poll_id, company_id, daterating, rating) VALUES
(1, 3, 6, 1, '2025-07-10', 5),
(2, 7, 6, 2, '2025-07-11', 4),
(3, 12, 6, 3, '2025-07-12', 5),
(4, 18, 6, 1, '2025-07-12', 3),
(5, 21, 6, 2, '2025-07-13', 4),
(6, 25, 6, 4, '2025-07-13', 5),
(7, 29, 6, 5, '2025-07-14', 3),
(8, 34, 6, 3, '2025-07-14', 5),
(9, 46, 6, 1, '2025-07-15', 4),
(10, 51, 6, 2, '2025-07-15', 5);

```

###### rates

```sql
INSERT INTO rates (customer_id, company_id, poll_id, daterating, rating) VALUES
(2, 1, 4, '2025-07-10', 5), 
(5, 2, 4, '2025-07-10', 4),
(9, 3, 4, '2025-07-11', 3),
(13, 4, 4, '2025-07-11', 5),
(18, 5, 4, '2025-07-12', 4),
(22, 1, 4, '2025-07-12', 2),
(29, 2, 4, '2025-07-13', 4),
(35, 3, 4, '2025-07-13', 5),
(41, 4, 4, '2025-07-14', 3),
(50, 5, 4, '2025-07-14', 4);

```

###### company_products

```sql
INSERT INTO companyproducts (company_id, product_id, unitmeasure_id) VALUES
(1, 1, 1),  
(1, 5, 2),  
(2, 2, 1),  
(2, 6, 3), 
(3, 3, 1),  
(3, 7, 1),  
(4, 4, 1),  
(4, 8, 1),  
(5, 9, 1), 
(5, 10, 1);

```

###### companyproduct_prices

```sql
INSERT INTO companyproduct_prices (company_id, product_id, price, start_date, end_date) VALUES
(1, 1, 750000, '2025-07-01', NULL),  
(1, 5, 35000, '2025-07-01', NULL),   
(2, 2, 3200000, '2025-07-01', NULL), 
(2, 6, 68000, '2025-07-01', NULL),   
(3, 3, 5100000, '2025-07-01', NULL), 
(3, 7, 210000, '2025-07-01', NULL),  
(4, 4, 950000, '2025-07-01', NULL),  
(4, 8, 520000, '2025-07-01', NULL),  
(5, 9, 870000, '2025-07-01', NULL),  
(5, 10, 390000, '2025-07-01', NULL); 

```

###### company_emails

```sql
INSERT INTO company_emails (company_id, email) VALUES 
(1, 'contacto@melodiasdelnorte.com'),
(2, 'ventas@ritmototal.com'),
(3, 'info@harmonyrecords.com'),
(4, 'soporte@sonartecolombia.com'),
(5, 'contacto@djfactorygroup.com');
```

###### favorites

los primeros 30 clientes tiene 2 empresas favoritas y los otros restante, solo 1.

```sql
INSERT INTO favorites (customer_id, company_id) VALUES
(1, 1), (1, 2),
(2, 2), (2, 3),
(3, 3), (3, 4),
(4, 4), (4, 5),
(5, 5), (5, 1),
(6, 1), (6, 3),
(7, 2), (7, 4),
(8, 3), (8, 5),
(9, 4), (9, 1),
(10, 5), (10, 2),
(11, 1), (11, 4),
(12, 2), (12, 5),
(13, 3), (13, 1),
(14, 4), (14, 2),
(15, 5), (15, 3),
(16, 1), (16, 5),
(17, 2), (17, 4),
(18, 3), (18, 2),
(19, 4), (19, 1),
(20, 5), (20, 3),
(21, 1), (21, 2),
(22, 2), (22, 3),
(23, 3), (23, 4),
(24, 4), (24, 5),
(25, 5), (25, 1),
(26, 1), (26, 3),
(27, 2), (27, 4),
(28, 3), (28, 5),
(29, 4), (29, 1),
(30, 5), (30, 2),
(31, 1),
(32, 2),
(33, 3),
(34, 4),
(35, 5),
(36, 1),
(37, 2),
(38, 3),
(39, 4),
(40, 5),
(41, 1),
(42, 2),
(43, 3),
(44, 4),
(45, 5),
(46, 1),
(47, 2),
(48, 3),
(49, 4),
(50, 5),
(51, 1),
(52, 2),
(53, 3),
(54, 4),
(55, 5),
(56, 1),
(57, 2),
(58, 3),
(59, 4),
(60, 5);

```



details_favorites

```sql
INSERT INTO details_favorites (favorite_id, product_id, category_id, name, detail) VALUES
(1, 1, 1, 'Guitarra Acústica Yamaha F310', 'Guitarra acústica de cuerpo completo con cuerdas de acero, ideal para principiantes y estudiantes.'),
(2, 2, 2, 'Batería Acústica Pearl Roadshow 5‑pz', 'Set completo de batería acústica 5 piezas con platillos incluidos, ideal para estudios o presentaciones en vivo.'),
(3, 3, 3, 'Saxofón Alto Yamaha YAS‑280', 'Saxofón alto de excelente calidad para estudiantes avanzados, con estuche rígido incluido.'),
(4, 4, 4, 'Teclado Casio CT‑X700', 'Teclado portátil de 61 teclas con polifonía de 48 notas, 600 tonos y 195 ritmos.'),
(5, 5, 5, 'Púas Fender Heavy (x12)', 'Paquete de 12 púas Fender de celuloide con grosor heavy, ideales para guitarristas de rock.'),
(6, 6, 6, 'Cable Instrumento Planet Waves 3 m', 'Cable profesional para guitarra o bajo de 1/4", longitud 3 metros, conectores niquelados.'),
(7, 7, 7, 'Estuche Rígido para Guitarra Clásica', 'Estuche protector tipo maleta para guitarra clásica, con interior acolchado y cerradura.'),
(8, 8, 8, 'Micrófono Shure SM58', 'Micrófono dinámico cardioide legendario para voz, resistente y confiable.'),
(9, 9, 9, 'Interfaz Focusrite Scarlett 2i2 Gen3', 'Interfaz de audio USB con dos entradas XLR/Jack y salidas balanceadas, ideal para home studio.'),
(10, 10, 10, 'Luz LED Par Chauvet DJ SlimPAR 56', 'Luz LED compacta de escenario con mezcla de colores RGB, ideal para eventos en vivo.'),
(11, 1, 1, 'Guitarra Acústica Yamaha F310', 'Guitarra acústica de cuerpo completo con cuerdas de acero, ideal para principiantes y estudiantes.'),
(12, 2, 2, 'Batería Acústica Pearl Roadshow 5‑pz', 'Set completo de batería acústica 5 piezas con platillos incluidos, ideal para estudios o presentaciones en vivo.'),
(13, 3, 3, 'Saxofón Alto Yamaha YAS‑280', 'Saxofón alto de excelente calidad para estudiantes avanzados, con estuche rígido incluido.'),
(14, 4, 4, 'Teclado Casio CT‑X700', 'Teclado portátil de 61 teclas con polifonía de 48 notas, 600 tonos y 195 ritmos.'),
(15, 5, 5, 'Púas Fender Heavy (x12)', 'Paquete de 12 púas Fender de celuloide con grosor heavy, ideales para guitarristas de rock.'),
(16, 6, 6, 'Cable Instrumento Planet Waves 3 m', 'Cable profesional para guitarra o bajo de 1/4", longitud 3 metros, conectores niquelados.'),
(17, 7, 7, 'Estuche Rígido para Guitarra Clásica', 'Estuche protector tipo maleta para guitarra clásica, con interior acolchado y cerradura.'),
(18, 8, 8, 'Micrófono Shure SM58', 'Micrófono dinámico cardioide legendario para voz, resistente y confiable.'),
(19, 9, 9, 'Interfaz Focusrite Scarlett 2i2 Gen3', 'Interfaz de audio USB con dos entradas XLR/Jack y salidas balanceadas, ideal para home studio.'),
(20, 10, 10, 'Luz LED Par Chauvet DJ SlimPAR 56', 'Luz LED compacta de escenario con mezcla de colores RGB, ideal para eventos en vivo.'),
(21, 1, 1, 'Guitarra Acústica Yamaha F310', 'Guitarra acústica de cuerpo completo con cuerdas de acero, ideal para principiantes y estudiantes.'),
(22, 2, 2, 'Batería Acústica Pearl Roadshow 5‑pz', 'Set completo de batería acústica 5 piezas con platillos incluidos, ideal para estudios o presentaciones en vivo.'),
(23, 3, 3, 'Saxofón Alto Yamaha YAS‑280', 'Saxofón alto de excelente calidad para estudiantes avanzados, con estuche rígido incluido.'),
(24, 4, 4, 'Teclado Casio CT‑X700', 'Teclado portátil de 61 teclas con polifonía de 48 notas, 600 tonos y 195 ritmos.'),
(25, 5, 5, 'Púas Fender Heavy (x12)', 'Paquete de 12 púas Fender de celuloide con grosor heavy, ideales para guitarristas de rock.'),
(26, 6, 6, 'Cable Instrumento Planet Waves 3 m', 'Cable profesional para guitarra o bajo de 1/4", longitud 3 metros, conectores niquelados.'),
(27, 7, 7, 'Estuche Rígido para Guitarra Clásica', 'Estuche protector tipo maleta para guitarra clásica, con interior acolchado y cerradura.'),
(28, 8, 8, 'Micrófono Shure SM58', 'Micrófono dinámico cardioide legendario para voz, resistente y confiable.'),
(29, 9, 9, 'Interfaz Focusrite Scarlett 2i2 Gen3', 'Interfaz de audio USB con dos entradas XLR/Jack y salidas balanceadas, ideal para home studio.'),
(30, 10, 10, 'Luz LED Par Chauvet DJ SlimPAR 56', 'Luz LED compacta de escenario con mezcla de colores RGB, ideal para eventos en vivo.');

```

company_phones

```sql
INSERT INTO company_phones (company_id, phone) VALUES
(1, '+57 3001234567'),         
(2, '+57 3012345678'),         
(3, '+1 2125557890'),         
(4, '+57 3123456789');         

```










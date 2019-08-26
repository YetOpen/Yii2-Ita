#Convalida dell'input

Come regola generale, non dovresti mai fidarti dei dati ricevuti dagli utenti finali e dovresti sempre convalidarli prima di farne uso.

Dato un modello popolato con input dell'utente, è possibile convalidare gli input chiamando la funzione yii\base\Model::validate(). La funzione resistuirà un valore booleano che indica se la convalida è andata a buon fine. In caso contrario, potresti ricevere un messaggio d'errore dalla proprietà yii\base\Model::$errors. Per esempio, 

	$model = new \app\models\ContactForm();
	
	// populate model attributes with user inputs
	$model->load(\Yii::$app->request->post());
	// which is equivalent to the following:
	// $model->attributes = \Yii::$app->request->post('ContactForm');
	
	if ($model->validate()) {
		// all inputs are valid
	} else {
		// validation failed: $errors is an array containing error messages
		$errors = $model->errors;
	}

### Dichiarazione delle regole

Affinché `validate()` funzioni davvero, è necessario dichiarare le regole di validità per gli attributi che si intende convalidare. Questo andrebbe fatto sovrascrivendo la funzione yii\base\Model::rules(). Il seguente esempio mostra come vengono dichiarate le regole di validità per `ContactForm`: 

	public function rules()
	{
		return [
			// the name, email, subject and body attributes are required
			[['name', 'email', 'subject', 'body'], 'required'],
			
			// the email attribute should be a valid email address
			['email', 'email'],
		];
	}

La funzione rules() restituirà un array di regole, ognuna delle quali è un array con la seguente formattazione:

	[
		// required, specifies which attributes should be validated by this rule.
		// For a single attribute, you can use the attribute name directly
		// without having it in an array
		['attribute1', 'attribute2', ...],
		
		// required, specifies the type of this rule.
		// It can be a class name, validator alias, or a validation method name
		'validator',
		
		// optional, specifies in which scenario(s) this rule should be applied
		// if not given, it means the rule applies to all scenarios
		// You may also configure the "except" option if you want to apply the rule
		// to all scenarios except the listed ones
		'on' => ['scenario1', 'scenario2', ...],
		// optional, specifies additional configurations for the validator object
		'property1' => 'value1', 'property2' => 'value2', ...
	]
	
Per ogni regola è necessario specificare almeno a quali attributi si applica e qual è il tipo di regola. È possibile specificare il tipo di regola in uno dei seguenti modi:

- lo pseudonimo di un validatore di base, come `required`, `in`, `date`, etc. Riferirsi a Core Validators per una lista completa di valdatori di base.

- il nome di una funzione di convalida nella classe del modello, o una funzione anonima. Per ulteriori informazioni riferirsi alla sottosezione Inline Validators.

- un nome della classe convalida completo. Per ulteriori informazioni, riferirsi alla sottosezione Standalone Validators.

Una regola può essere usata per convalidare uno o più attributi e un attributo può essere convalidato da una o più regole. Una regola può essere applicata in un certo scenario solo se viene specificata l'opzione `on`. Se non viene specificata, significa che la regola verrà applicata a tutti gli scenari.

Quando la funzione `validate()` viene chiamata, esegue i seguenti passaggi per eseguire la convalida:

1. Determinare quali attributi devono essere convalidato, ottenendo l'elenco di attributi da yii\base\Model::scenarios() utilizzando lo scenario corrente. Questi attributi vengono chiamati *attributi attivi*.
2. Determinare quali regole di convalida devono essere usate, ottenendo l'elenco di regole da yii\base\Model::rules() utilizzando lo scenario corrente. Queste regole vengono chiamate *regole attive*.
3. Utilizzare ogni regola attiva per convalidare ogni attributo attivo associato alla regola. Le regole di convalida vengono valutate nell'ordine in cui sono elencate.

Secondo questi passaggi di convalida, un attributo verrà convalidato se e solo se è un attributo attivo dichiarato in `scenarios()` ed è associato ad una o più regole attive dichiarate in `rules()`.

*Nota: è comodo dare alle regole dei nomi, esempio*
	
	public function rules()
	{
			return [
				// ...
				'password' => [['password'], 'string', 'max' => 60],
			];
		}

*È possibile usarla in un modello secondaro:*

	public function rules()
	{
		$rules = parent::rules();
		unset($rules['password']);
		return $rules;
	}
	
###Personalizzazione messaggi d'errore

La maggior parte dei convalidatori hanno messaggi d'errore preimpostati che verranno aggiunti al modello in fase di validzione quando i suoi attributi falliscono la convalidazione. Per esempio, la convalida richiesta aggiungerà un messaggi "Username cannot be blank." a un modello quando l'attributo `username` fallisce nel momento in cui viene utilizzata questa regola di convalida.

 È possibile personalizzare il messaggio d'errore di una regola specificando la proprietà `message` quando viene dichiarata la regola, come segue, 
 
 	public function rules()
 	{
 		return [
 			['username', 'required', 'message' => 'Please choose a username.'],
 		];
 	}
 	
Alcune convalide possono supportare un ulteriore messaggio d'errore che descrive specificatamente la possibile causa del fallimento della convalida. Per esempio, il supporto di convalida tooBig e tooSmall di un numero descrive il fallimento della convalida quando il valore convalidato è rispettivamente troppo grande o troppo piccolo.  È possibile configurare questo messaggio d'errore come la configurazione di altre proprietà dei convalidatori in una regola di convalida.

###Eventi di convalidazione###

Quando yii\base\Model::validate() viene chiamato, richiamerà due funzioni che è possibile ignorare per personalizzare il processo di convalida:

- yii\base\Model::beforeValidate(): l'implemntazione predefinita attiverà un evento  yii\base\Model::EVENT_BEFORE_VALIDATE. È possibile ignorare questa funzione o rispondere a questo evento per eseguire alcuni lavori di preelaborazione (ad esempio normalizzando i dati di input) prima che avvenga la convalida. La funzione dovrebbe restituire un valore booleano che indica se la convalida può procedere o no. 

- yii\base\Model::afterValidate(): l'implemntazione predefinita attiverà un evento  yii\base\Model::EVENT_AFTER_VALIDATE. È possibile ignorare questa funzione o rispondere a questo evento per eseguire alcuni lavori di postelaborazione dopo il completamento della convalida.

### Convalida condizionale

Per convalidare gli attributi solo quando determinate condizioni sono applicate, per esempio la convalida di un attributo dipende dal valore di un altro attributo che consente l'utilizzo della proprietà when per definire tale condizione. Per esempio, 

	 ['state', 'required', 'when' => function($model) {
	 	return $model->country == 'USA';
	 }]

La proprietà when prende un PHP richiamabile con il seguente codice:

	/**
	 * @param Model $model the model being validated
	 * @param string $attribute the attribute being validated
	 * @return bool whether the rule should be applied
	 */
	function ($model, $attribute)

Se è necessario supportare anche la convalida condizionale del client-side, è possibile configurare la proprietà whenClient che prende una stringa rappresentante una funzione JavaScript che riporta un valore se la regola viene applivata o no. Per esempio, 
	
	 ['state', 'required', 'when' => function ($model) {
	 	return $model->country == 'USA';
	 }, 'whenClient' => "function (attribute, value) {
	 	return $('#country').val() == 'USA';
	 }"]
	 
### Filtraggio dati

Gli insermenti dell'utente spesso necessitano di essere filtrati o processati. Per esempio, è possibile voler tagliare gli spazi attorno all'input `username`. È possibile usare le regole di convalide per raggiungere questo obiettivo. 

Il seguente esempio mostra come è possibile ritagliare gli spazi di input e trasformare gli input vuoti in null usando i validatori di base *trim* e *default*:
 
	return [
		[['username', 'email'], 'trim'],
		[['username', 'email'], 'default'],
	];

È possibile anche utilizzare il più generale filtro di convalida per eseguire il filtraggio di dati più complessi. 

Come puoi vedere, queste reole di convalida non convalidao davvero gli inserimenti. Infatti, elaboreranno i valori e li salveranno nuovamente negli attributi in fase di convalida.

Un'elaborazione completa degli input dell'utente è mostrata nel seguente codice d'esempio, che garantirà solo i valori interi immagazzinati in un attributo: 

	['age', 'trim'],
	['age', 'default', 'value' => null],
	['age', 'integer', 'min' => 0],
	['age', 'filter', 'filter' => 'intval', 'skipOnEmpty' => true],

Il codice soprastante eseguirà le seguenti operazioni sull'inserimento:

1. Tagliare gli spazi bianchi dal valore inserito.

2. Assicurarsi gli input vuoti siano memorizzati come `null` nel database; c'è differenza tra un valore "non impostato" e un effettivo valore 0. se `null` non è ammesso è possibile impostare un valore preimpostato qui.

3. Convalidare che i valore sia un numero intero maggiore di 0 se non è vuoto. Convalidatori normali hanno $skipOnEmpty impostato su `true`.

4. Assicurarsi che il valore sia di tipo intero, per esempio lanciare una stringa '42' nell'intero 42. Qui impostiamo $skipOnEmpty su `true`, che è preimpostato `false` dal filtro di convalida.

###Manipolazione degli spazi vuoti

Quando i dati inseritivengono inviati da un modulo HTML, è spesso necessario assegnare alcuni valori di default agli input se sono vuoti. È possibile farlo usando i convalidatori di default. Per esempio, 

	return [
		// set "username" and "email" as null if they are empty
		[['username', 'email'], 'default'],
		
		// set "level" to be 1 if it is empty
		['level', 'default', 'value' => 1],
	];
Di default, un inserimento è considerato vuoto se il suo valore è una stringa vuota, un array vuoto o un `null`. Èpossibile personalizzare la logica di rilevazione del vuoto predefinito configurando la proprietà You may customize the default empty detection logic con un richiamo PHP. Per esempio, 

	 ['agree', 'required', 'isEmpty' => function ($value) {
	 	return empty($value);
	 }]
	 
*Nota: la maggior parte dei convalidatori non gestisce gli inserimenti vuoti se la loro proprietà yii\validators\Validator::$skipOnEmpty risulta predefinita sul valore `true`. Semplicemente verrano saltati durante la convalise se i loro attribui associati ricevono degli inserimenti vuoti. Tra i convalidatori di base, solo i convalidatori `captcha`, `default`, `filter`, `required`, e `trim` gestiscono gli inserimenti vuoti.*

###Convalida ad Hoc

A volte è necessario fare una convalida ad Hoc per valori che non sono vincolati a nessun modello. 

Se è necessario eseguire un tipo di convalida (per esempio convalidare un indirizzo email), è possibile richiamare la funzione validate() del convalidatore desiderato, come segue:

	$email = 'test@example.com';
	$validator = new yii\validators\EmailValidator();
	
	if ($validator->validate($email, $error)) {
		echo 'Email is valid.';
	} else {
		echo $error;
	}
	
*Nota: non tutti i convalidatori supportano questo tipo di convalida. Un esempio è il convalidatore di base *unique* che è stato realizzato per lavorare con un solo modello.*

*Nota: la proprietà  yii\base\Validator::skipOnEmpty viene utilizzata per convalidare solo yii\base\Model. Non ha effetto se usata senza modello.

Se è necessario eseguire convalidazioni multiple contro diverse regole, è possibile usare  yii\base\DynamicModel che supporta dichiarazioni al volo di attributi e regole. Il suo utilizzo è come segue:

	public function actionSearch($name, $email)
	{
		$model = DynamicModel::validateData(['name' => $name, 'email' => $email], [
			[['name', 'email'], 'string', 'max' => 128],
			['email', 'email'],
		]);
		
		if ($model->hasErrors()) {
			// validation fails
		} else {
			// validation succeeds
		}
	}

La funzione yii\base\DynamicModel::validateData() crea un'istanza di `DynamicModel`, definisce gli attributi usando i dati ricevuti (`name` e `email` in questo esempio), e poi chiama  yii\base\Model::validate() con le regole date. 

In alternativa, è possibile usare le seguenti sintassi più "classiche" per eseguire ad Hoc le convalide di dati: 

	public function actionSearch($name, $email)
	{
		$model = new DynamicModel(['name' => $name, 'email' => $email]);
		$model->addRule(['name', 'email'], 'string', ['max' => 128])
			->addRule('email', 'email')
			->validate();
			
		if ($model->hasErrors()) {
			// validation fails
		} else {
			// validation succeeds
		}
	}

Dopo la convalida, è possibile controllare se la convalida ha avuto successo o no chiamando la funzione hasErrors(), e poi ricevere l'errore di convalida dalla proprietà *errors*, come si farebbe con un modello normale. È possibile anche accedere agli attributi dinamici attraverso l'istanza modello, per esempio `$model->name` e `$model->email`.

###Creare convalidatori 

Oltre a usare i convalidatori di vase include nel rilascio di Yii, è anche possibile creare i propri convalidatori. È possibile creare convalidatori inline o convalidatori standalone. 

####Convalidatori inline

Un convalidatore inline è definito in termini di una funzione modello o anonima. il codice di questa funzione è:

	/**
	 * @param string $attribute the attribute currently being validated
	 * @param mixed $params the value of the "params" given in the rule
	 * @param \yii\validators\InlineValidator $validator related InlineValidator instance.
	 * This parameter is available since version 2.0.11.
	 */
	function ($attribute, $params, $validator)
	
Se un attributo fallisce la convalida, la funzione richiamerebbe yii\base\Model::addError() per salvare il messaggio d'errore nel modllo di modo che possa essere ricavato dopo da presentare all'utente finale.

Sotto alcuni esempi:

	use yii\base\Model;
	
	class MyForm extends Model
	{
		public $country;
		public $token;
		
		public function rules()
		{
			return [
				// an inline validator defined as the model method validateCountry()
				['country', 'validateCountry'],
				
				// an inline validator defined as an anonymous function
				['token', function ($attribute, $params, $validator) {
					if (!ctype_alnum($this->$attribute)) {
						$this->addError($attribute, 'The token must contain letters or digits.');
					}
				}],
			];
		}
		
		public function validateCountry($attribute, $params, $validator)
		{
			if (!in_array($this->$attribute, ['USA', 'Indonesia'])) {
				$this->addError($attribute, 'The country must be either "USA" or "Indonesia".');
			}
		}
	}
	
*Nota: fino alla versione 2.0.11 è possibile usare yii\validators\InlineValidator::addError() per l'aggiunta di errori. In questo modo il messaggio d'errore può essere formattato subito usando yii\i18n\I18N::format(). Utilizzare `{attribute}` e `{value}` nei messaggi d'errore per riferirsi all'attributo label (non è necessario ottenerlo manualmente e di conseguenza all'attributo value:*
	
	$validator->addError($this, $attribute, 'The value "{value}" is not acceptable for {attribute}.'); 
	
*Nota: di default, convalidatori inline non possono essere applicati se i loro attributi associati ricevono un input vuoto o se hanno già fallito alcune regole di convalida. Se vuoi assicurarti che una regola sia sempre applicabile, è possibile configurare la proprietà skipOnEmpty e/o skipOnError perché sia `false` nella dichiarazione delle regole. Per esempio:*

	[
		['country', 'validateCountry', 'skipOnEmpty' => false, 'skipOnError' => false],
	]
	
####Convalidatori standalone

Un convalidatore standalone è una classe che estende yii\validators\Validator o la sua classe figlio. È possibile implementare la sua convalida sovrascrivendo la funzione yii\validators\Validator::validateAttribute(). Se un attributo fallisce la convalda, chiamare yii\base\Model::addError() per salvare il messaggio d'errore nel modello, come è stato fatto con il convalidatore inline.

Per esempio, i convalidatori inline sopra posso essere spostati nella nuova classe [[components/validators/CountryValidator]]. In questo caso è possibile usare yii\validators\Validator::addError() per impostare la personalizzazione del messaggio per il modello.

	namespace app\components;
	
	use yii\validators\Validator;
	
	class CountryValidator extends Validator
	{
		public function validateAttribute($model, $attribute)
		{
			if (!in_array($model->$attribute, ['USA', 'Indonesia'])) {
				$this->addError($model, $attribute, 'The country must be either "{country1}" or "{country2}".', ['country1' => 'USA', 'country2' => 'Indonesia']);
			}
		}
	}
	
Se desideri che il tuo convalidatore supporti la convalida di un valore senza un modello, dovresti anche sovrascrivere yii\validators\Validator::validate(). Puoi ance sovrascrivere yii\validators\Validator::validateValue() invece di `validateAttribute()` e `validate()` perché di default queste due ultime funzioni sono implementate richiamando `validateValue()`.

Di seguito è riportato un esempio di come è possibile utilizzare una classe di convalida nel tuo modello.

	namespace app\models;
	
	use Yii;
	use yii\base\Model;
	use app\components\validators\CountryValidator;
	
	class EntryForm extends Model
	{
		public $name;
		public $email;
		public $country;
		
		public function rules()
		{
			return [
				[['name', 'email'], 'required'],
				['country', CountryValidator::className()],
				['email', 'email'],
			];
		}
	}
### Convalida di più attributi

A volte i convalidatori coinvolgono più attributi. Considera il seguente modulo:

	class MigrationForm extends \yii\base\Model
	{
		/**
		 * Minimal funds amount for one adult person
		 */
		const MIN_ADULT_FUNDS = 3000;
		/**
		 * Minimal funds amount for one child
		 */
		const MIN_CHILD_FUNDS = 1500;
		
		public $personalSalary;
		public $spouseSalary;
		public $childrenCount;
		public $description;
		
		public function rules()
		{
			return [
				[['personalSalary', 'description'], 'required'],
				[['personalSalary', 'spouseSalary'], 'integer', 'min' => self::MIN_ADULT_FUNDS],
				['childrenCount', 'integer', 'min' => 0, 'max' => 5],
				[['spouseSalary', 'childrenCount'], 'default', 'value' => 0],
				['description', 'string'],
			];
		}
	}
	
#### Creare convaldatori

Diciamo che dobbiamo verificare se il reddito familiare è sufficiente per i tuoi bambini. È possibile creare un convalidatore inline `validateCildrenFounds` per questo che si azionerà solo se il `childrenCount` è maggiore di 0.

Si noti che non è possibile utilizzare tutti gli attributi convalidati ([`personalSalary`, `spouseSalary`, `childrenCount`]) quando si allega il convalidatore. Questo perché il convalidatore stesso si azionerà per ogni attributo (3 volte in totale) e dobbiamo eseguirlo una sola volta per tutto il set di attributi.

È possibile usare uno di questi attributi al loro posto (o o usare quello che sembra il più adeguato):

	['childrenCount', 'validateChildrenFunds', 'when' => function ($model) {
		return $model->childrenCount > 0;
	}],

L'implementazione di `validateChildrenFounds potrebbe essere così:

	public function validateChildrenFunds($attribute, $params)
	{
		$totalSalary = $this->personalSalary + $this->spouseSalary;
		// Double the minimal adult funds if spouse salary is specified
		$minAdultFunds = $this->spouseSalary ? self::MIN_ADULT_FUNDS * 2 : self::MIN_ADULT_FUNDS;
		$childFunds = $totalSalary - $minAdultFunds;
		if ($childFunds / $this->childrenCount < self::MIN_CHILD_FUNDS) {
			$this->addError('childrenCount', 'Your salary is not enough for children.');
		}
	}

È possibile ignoare il parametro `$attribute` perché non è relazionato ad un solo attributo.

#### Aggiungere errori

L'aggiunta di errori in caso di più attributi può variare a seconda del design del modulo desiderato:

-Selezionare il campo più rilevante secondo te e aggiugere l'errore al suo attributo:

	`$this->addError('childrenCount', 'Your salary is not enough for children.');`

-Selezionare gli attributi più rilevanti o tutti gli attributi e aggiungergli lo stesso messaggio d'errore. È possibile memorizzare il messaggio in variabile separate rima di passarlo a `addError` per mantenere il codice pulito. 

	$message = 'Your salary is not enough for children.';`
	$this->addError('personalSalary', $message);`
	$this->addError('wifeSalary', $message);`
	$this->addError('childrenCount', $message);`
oppure utilizzare un loop

	$attributes = ['personalSalary', 'wifeSalary', 'childrenCount'];
	foreach ($attributes as $attribute) {
		$this->addError($attribute, 'Your salary is not enough for children.');
	}

-Aggiugere un normale errore( non relazionato a un particolare attributo). È possibile usare un attributo inesistente per aggiungere errori, per esempio `*`, perché l'esistenza di un attributo non è controllata a questo punto.

	$this->addError('*', 'Your salary is not enough for children.');

Come risultato, non vedremo messaggi di errore vicino ai campi del modulo. Per visualizzarli, possiamo includere il riepilogo degli errori i vista:

	<?= $form->errorSummary($model) ?>
	
*Nota: Creare convalidatori che convalidano più attributi contemporaneamente è ben descritta su community cookbook.

###Convalida client-side

La convalida client-side basata su JavaScript è auspicabile quando gli utenti finali fornisco input tramite moduli HTML, perchè consente agli uteni di trovare l'errore più velocemente e offre quindi una migliore esperienza all'utente. È possibile usare o implementare un convalidatore che supporti la convalida client-side oltre alla convalida server-side.

*Info: sebbene la convalida client-side è auspicabile, non è indispensabile. Il suo scopo principale è fornire agli utenti un'esperienza migliore. Come con i dati di input provenienti dagli utenti finali, non dovresti mai fidarti della convalida client-side. Per questo motivo, dovresti sempre eseguire la convalida server-side chiamando yii\base\Model::validate(), come descritto nella precedente sottosezione.*

#### Usare la convalida Client-Side

Molti convalidatori di base supportano la convalida client-side out-of-the-box. Tutto quello che c'è da fare è solo usare yii\widgets\ActiveForm per costruire il tuo modulo HTML. Per esempio, `LoginForm` sotto dichiara due regole: la prima usa il convalidare di base *required* che è supportato sia dal client-side che dal server-side; l'altro usa il convalidatore inlne `validatePassword` che è solo supportato dal server-side.

	namespace app\models;
	
	use yii\base\Model;
	use app\models\User;
	
	class LoginForm extends Model
	{
		public $username;
		public $password;
		
		public function rules()
		{
			return [
				// username and password are both required
				[['username', 'password'], 'required'],
				
				// password is validated by validatePassword()
				['password', 'validatePassword'],
			];
		}
		
		public function validatePassword()
		{
			$user = User::findByUsername($this->username);
			
			if (!$user || !$user->validatePassword($this->password)) {
				$this->addError('password', 'Incorrect username or password.');
			}
		}
	}

Il modulo HTML creato dal seguente codice contiene due campi di inserimento `username` e `password`. Se invii il modulo senza inserire niente, troverai che i messaggi d'errore che richiedono di inserire qualcosa appaiono subito senza alcunacomunicazione con il server.

	<?php $form = yii\widgets\ActiveForm::begin(); ?>
		<?= $form->field($model, 'username') ?>
		<?= $form->field($model, 'password')->passwordInput() ?>
		<?= Html::submitButton('Login') ?>
	<?php yii\widgets\ActiveForm::end(); ?>

Dietro le quinte,  yii\widgets\ActiveForm leggerà le regole di convalida dichiarate nel modello e genererà un codice JavaScript appropriato per convalidatori che supporti la convalida client-side. Quando un utente cambia il valore di un campo di inserimento o invia il modulo, la convalida client-side JavaScript verrà attivata.

Se vuoi spegnere la convalida client-side competamente, è possibile configurare la proprietà yii\widgets\ActiveForm::$enableClientValidation su `false`. È anche possibile spegnere la convalida client-side di campi di inserimento singoli cofigurando la loro proprietà yii\widgets\ActiveField::$enableClientValidation su `false`. QUando `enableClientValidation è configurato ad entrambi i livelli dei campi di input e i livelli di modulo, il primo prende la precedenza. 
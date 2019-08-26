#Creazione moduli
###moduli basati su ActiveRecord: ActiveForm


Il modo primario di usare moduli in Yii è attraverso  yii\widgets\ActiveForm. Questo approccio dovrebbe essere preferito quando il modulo è basato su un modello. Inoltre, esistono alcuni metodi utili in yii\helpers\Html che sono tipicamente usati per aggiungere bottoni e testo guida per ogni modulo.
 
 	Suggerimento: se i campi del modulo sono differenti dalle colonne del database o ci sono formattazione e logica che sono specifici di quel modulo, è preferibile creare e separare modelli estesi da yii\base\Model.
 Nel seguente esempio, mostreremo come modelli generici possano essere usati per un modulo d'accesso:
 
	<?php
	
	class LoginForm extends \yii\base\Model
	{
		public $username;
		public $password;
		
		public function rules() 
		{
			return [
				// define validation rules here
			];
		}
	}
Nel controller, passeremo un'esempio di come appare il modello sopra dove il widget ActiveForm viene usato per visualizzare il modulo:

	<?php
	use yii\helpers\Html;
	use yii\widgets\ActiveForm;

	$form = ActiveForm::begin([
		'id' => 'login-form',
		'options' => ['class' => 'form-horizontal'],
	]) ?>
		<?= $form->field($model, 'username') ?>
		<?= $form->field($model, 'password')->passwordInput() ?>
		
		<div class="form-group">
			<div class="col-lg-offset-1 col-lg-11">
				<?= Html::submitButton('Login', ['class' => 'btn btn-primary']) ?>
			</div>
		</div>
	<?php ActiveForm::end() ?>
	
	
## Racchiudere con begin() e end()

Nel codice sopra, ActiveForm::begin() non crea solo un modulo istanza, ma indica anche l'inizio del modulo. Tutti i contenuti posizionati tra ActiveForm::begin() e ActiveForm::end() verranno racchiusi dentro il tag HTML `<form>`. Come con qualsiasi widget, è possibile specificare alcune opzioni su come il widget dovrebbe essere configurato passando da un array al metodo `begin`. In questo caso, una classe CSS extra e un ID identificativo vengono passati per essere usati nell'apertura del tag `<form>`. Per tutte le impostazioni disponibili, perfavore riferirsi alla documentazione API di yii\widgets\ActiveForm.

###ActiveField

Per creare un elemento modulo in un modulo, insieme all'etichetta dell'elemento e a qualsiasi convalida JavaScript applicabile,  viene richiamata la funzione ActiveForm::field(), che restituisce un'istanza di yii\widgets\ActiveField. Quando il risultato di questa funzione viene ripetuto direttamente, il risulatato è un normale input (text). Per personalizzare l'output, è possibile incatenare una funzione aggiuntiva di ActiveField con questa chiamata: 

	// a password input
	<?= $form->field($model, 'password')->passwordInput() ?>
	// adding a hint and a customized label
	<?= $form->field($model, 'username')->textInput()->hint('Please enter your name')->label('Name') ?>
	// creating a HTML5 email input element
	<?= $form->field($model, 'email')->input('email') ?>
	
Questo codice creera tutti i tag `<label>`, `<input>` e altri secondo il modello definito dal campo del modulo. Il nome del campo di input viene determinato automaticamente dal nome del modulo del modello e dal nome attribuito. Per esempio, il nome per il campo di input per l'attributo `username`nell'esempio sopra sarà `LoginForm[username]`. Questa regola di denominazione comporterà un array di tutti gli attributi per il modulo di input che sarà disponibile in `$_POST['LoginForm'] sul lato del server. 

	Suggerimento: se si dispone di un solo modello in un modulo e si desidera semplificare i nomi di input, è possibile saltare la parte dell'array utilizzando la funzione formName() del modello per ottenere una stringa vuota. Questo può essere utile per filtrare modelli utilizzati nel GridView per creare ottimi URLs.
	
L'attributo del modello può essere specificato in modi più sofisticati. Per esempio, quando un attributo può assumere un valore di array, nel momento in cui si caricano più file o si selezionano più elementi, è possibile specificarlo aggiungendo  `[]` al nome dell'attributo: 

	// allow multiple files to be uploaded:
	echo $form->field($model, 'uploadFile[]')->fileInput(['multiple'=>'multiple']);

	// allow multiple items to be checked:
	echo $form->field($model, 'items[]')->checkboxList(['a' => 'Item A', 'b' => 'Item B', 'c' => 'Item C']);

Prestare attenzione quando si denomina l'elemento di un modulo come pulsante di inivio. Secondo il jQuery Documentation esistono alcuni nomi riservati che possono causare conflitti:

	I moduli e i loro elementi secondari non devono utilizzare nomi di input o ID in conflitto con le proprietà di un modulo, come ad esempio `submit`, `lenght`, o `method`. Nomi conflittuali posso causare errori di confusione. Per un elenco completo delle regole e per verificare il markup per questi problemi, guarda DOMLint.
	
Inoltre si possono aggiungere tag HTML al modulo attraverso HTML plain o usando i metodi provenienti da Html-helper class come è stato fatto nell'esempio sopra con Html::submitButton(). 

	Suggerimento: Se stai usando Twitter Bootstrap CSS nella tua applicazione potresti voler usare yii\bootstrap\ActiveForm invece di yii\widgets\ActiveForm. Il primo si estende al secondo e utilizza stili specfici di Bootstrap quando genera campi di input del modulo.
	
<e>

	Suggerimento: Per assegnare uno stile ai campi obbligatori con asterischi, è possibile utilizzare il seguente CSS:
			div.required label.control-label:after {
    				content: " *";
    				color: red;`
			}


###Creazione liste
Esistono tre tipi di liste:

- menu a tendina

- elenchi radio

- elenchi checkbox

Per creare una litsa, bisogna preparare l'oggetto. Può essere fatto manualmente: 

	$items = [
    	1 => 'item 1', 
    	2 => 'item 2'
	]

o mediante recupero dal BD:

	$items = Category::find()
        ->select(['label'])
        ->indexBy('id')
        ->column();

Questi `$items` devono essere elaborati dai diversi widget della lista. Il valore del campo del modulo (e l'elemento attivo corrente) verrà automaticamente impostato dal valore corrente dell'attributo `$model`.

**Creare un menù a tendina**
È possibile usare la funzione ActiveField yii\widgets\ActiveField::dropDownList() per creare un menù a tendina:

	/* @var $form yii\widgets\ActiveForm */
	
	echo $form->field($model, 'category')->dropdownList([
			1 => 'item 1', 
			2 => 'item 2'
		],
		['prompt'=>'Select Category']
	);

**Creare un elenco radio**
È possibilre usare la funzione ActiveField yii\widgets\ActiveField::radioList() per creare un elenco radio:

	/* @var $form yii\widgets\ActiveForm */
	
	echo $form->field($model, 'category')->radioList([
		1 => 'radio 1', 
		2 => 'radio 2'
	]);

**Creare un elenco Checkbox**
È possibile usare la funzione ActiveField yii\widgets\ActiveField::checkboxList() per creare un elenco checkbox:

	/* @var $form yii\widgets\ActiveForm */
	
	echo $form->field($model, 'category')->checkboxList([
		1 => 'checkbox 1', 
		2 => 'checkbox 2'
	]);
	
###Lavorare con Pjax

Il widget Pjax permette di aggiornare  una determinata sezione della pagina anzichè ricaricare l'intera pagina. Si può utilizzare per aggiornare solo il modulo e ricaricare il suo contenuto dopo l'invio.

È possibile configurare `$formSelector` per specificare quale invio del modulo può attivare Pjax. 

	use yii\widgets\Pjax;
	use yii\widgets\ActiveForm;
	
	Pjax::begin([
		// Pjax options
	]);
		$form = ActiveForm::begin([
			'options' => ['data' => ['pjax' => true]],
			// more ActiveForm options
		]);
		
			// ActiveForm content
		
		ActiveForm::end();
	Pjax::end();
	
<e>

	Suggerimento: Presatre attenzione ai collegamenti all'interno del widget Pjax poiché anche la risposta verrà visualizzata all'interno del widget. Per evitare ciò fai uso dell'attributo HTML `data-pjax="0"`.
	
#####Valori in pulsanti di invio e caricamento file

Esistono errori conosciuti nell'uso di `jQuery.serializeArray()` quando si rapporta con file e valori in pulsanti di invio che non possono essere risolti e che saranno invece a favore della classe `FormData` introdotta in HTML5.

Questo significa che l'unico supporto ufficiale per file e l'invio dei valori dei pulsanti con ajax o l'utilizzo del widget Pjax dipende dal supporto del browser per la classe `FormData`.

###Ulteriori letture

La sezione successiva 'Convalida dell'input' gestisce la convalida dei dati del modulo inviato sul server-side nonché la convalida ajax e client-side. 

Per leggere a proposito dell'uso più complesso dei moduli, potresti voler guardare le seguenti sezioni:

-  Raccolta di input tabulari per la raccolta di dati per più modelli dello stesso tipo.

-  Ottenere dati per più modelli per la gestione di più modelli diversi nella stessa forma.

-   Caricamento dei file su come utilizzare i moduli per il caricamento dei file.
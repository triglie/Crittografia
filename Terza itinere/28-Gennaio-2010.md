<h1 style="text-align:center">Prova in Itinere del 28 Gennaio 2010</h1>

### 1) Dimostrare che il problema BSquare non può essere più difficile del problema computazionale Diffie Hellman in $G$. In altre parole, si dimostri che, se esiste un avversario B capace di risolvere il problema CDH in $G$, tale avversario può essere sfruttato per risolvere il problema BSquare in $G$ e $G_T$ . (VEDI COMPITO)

Sia $B$ un avversario capace di risolvere il problema CDH in $G$. Si consideri quindi il seguente avversario $A$ che lo sfrutta:

```pseudocode
A(X): /* X = g**x */
	Y <- B(X, X) /* se B è bravo: Y = g**(x*x) = g**(x**2) */
	return e(g, Y) /* e(g, g**(x**2)) = e(g, g)**(x**2) */
```

$$
Pr[ESP_{G,g}^{BSquare}(A) = 1] = Pr[ESP_{G,g}^{cdh}(B) = 1 \\
Adv_{G,g}^{BSquare}(A) \ge Adv_{G,g}^{cdh}(B)
$$

Il vantaggio di questo avversario in senso **BSquare** è maggiore o uguale al vantaggio di B in senso **cdh**.

### 2) Introdurre e definire formalmente il concetto di indistinguibilità ind-id-cpa per cifrari basati sull’identità.

Sia $\mathcal{IBE} = (\mathcal{Setup}, \mathcal{KeyDer}, \mathcal{Enc}, \mathcal{Dec})$ un cifrario basato sull'identità. Si consideri $A$ un avversario che attacca questo cifrario in senso **ind-id-cpa**. $A$ riceve in input, oltre la definizione del cifrario, anche la chiave pubblica e la possibilità di consultare due oracoli: uno di cifratura, che cifra, utilizzando un **id** deciso da $A$, un messaggio secondo una funzione $LR(x_0, x_1, b) = \begin{cases} x_0 & b = 0 \\ x_1 & b= 1 \end{cases}$. Il secondo genera la chiave corrispondente all'**id** che riceve in input. L'avversario non può dare in input al secondo oracolo l'**id** scelto in partenza. L'obiettivo di $A$ è determinare in che esperimento si trova tramite l'output del bit corrispondente.

```pseudocode
ESP_{IBE}^{ind-id-cpa-1}(A):
	(pk, msk) <- Setup()
	b <- A^{ENC_{pk}(id*, LR(., ., 1)), KeyDer_{msk}(.)}
	if A cheats:
		return 0
	return b

ESP_{IBE}^{ind-id-cpa-0}(A):
	(pk, msk) <- Setup()
	b <- A^{ENC_{pk}(id*, LR(., ., 0)), KeyDer_{msk}(.)}
	if A cheats:
		return 0
	return b
```

Il vantaggio di $A$ in senso **ind-id-cpa** è così definito:
$$
Adv^{ind-id-cpa}(A) = | Pr[ESP^{ind-id-cpa-1}(A) = 1] - Pr[ESP^{ind-id-cpa-0}(A) = 1] |
$$
Il cifrario $\mathcal{IBE}$ è sicuro in senso **ind-id-cpa** se il vantaggio è prossimo a 0 per ogni avversario polinomialmente limitato.

### 3) Dimostrare che il seguente cifrario non è sicuro in senso IND-CCA.

???

### 4) Definire formalmente il concetto di sicurezza (ovvero non falsificabilità relativamente ad attacchi a messaggio scelto) per schemi di firma digitale.

Sia $\mathcal{DS}=(\mathcal{KeyGen}, \mathcal{Sign}, \mathcal{VF})$ uno schema di firma digitale e sia $A$ un avversario tenta di attaccare $\mathcal{DS}$ in senso **uf-cma**.

Ad $A$ viene data la possibilità di consultare un oracolo di firma, in grado di produrre il tag relativo ad un messaggio scelto da $A$. Il compito dell'attaccante è quello di produrre una coppia (tag, messaggio) che risulti valida alla funzione $\mathcal{VF}$ con un messaggio che non ha precedentemente dato in input al primo oracolo. Si noti che $A$ non ha bisogno di un oracolo di verifica poiché possiede tutte le informazioni necessarie per eseguire autonomamente la funzione $\mathcal{VF}$

Il tutto può essere formalizzato con il seguente esperimento:

```pseudocode
ESP_{DF}^{uf-cma}:
	(pk, sk) <- KeyGen()
	(mess, tag) <- A^{Sign_{sk}(.)}(pk)
	if A cheats:
		return 0
	return VF_{pk}(mess, tag) == 1
```

Il vantaggio di $A$ è così definito:
$$
ADV_{\mathcal{DF}}^{uf-cma}(A) = Pr[ESP_{\mathcal{DF}}^{uf-cma}(A) = 1]
$$
Lo schema di firma digitale è sicuro in senso **uf-cma** se tale vantaggio è prossimo a zero per ogni avversario polinomialmente limitato.

### 5) Si dimostri che il seguente schema di firma non è sicuro. (VEDI COMPITO)

Definito $d = e^{-1}$, si noti che $\mathcal{Sign}(m_1) = \sigma_1 = A^{dm_1}$. Ne segue che $2\sigma_1 = A^{d2m_1}$.

Si può quindi costruire il seguente avversario:

```pseudocode
A(vk):
	m1 <-R- M
	s1 <- O_{Sign}(m1)
	m2 <- 2 * m1
	s2 <- s1 ** 2
	return (m2, s2)
```

Per quanto detto in precedenza, questo avversario ha un vantaggio in senso **uf-cma** pari ad 1

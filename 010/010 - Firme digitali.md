<h1 style="text-align: center">Firme digitali</h1>

[TOC]

Una firma digitale è l'equivalente digitale delle firme convenzionali. Il concetto potrebbe richiamare la Message Authentication, solo che in questo caso abbiamo una struttura asimmetrica, con una chiave privata in grado di apporre la firma ed una pubblica in grado di verificarne la validità.

## Firma Digitale vs. MAC

La firma digitale non è, in generale, ripudiabile. Il problema è che nel caso simmetrico del MAC chi autentica e chi verifica agli occhi del programma è la stessa entità. In altre parole, chiunque è in grado di validare la firma, è anche in grado di apporla. Tale limitazione è superata dalla firma digitale, che introduce due ruoli distinti.

## Definizione formale

Uno schema di firma digitale è una tripla $\mathcal{DS}=(\mathcal{KeyGen}, \mathcal{Sign}, \mathcal{VF})$ così definita:

- $\mathcal{KeyGen}$: funzione randomizzata che non accetta input e produce una coppi di chiavi **pk**, **sk**
- $\mathcal{Sign}$: funzione randomizzata o a stati che prende in input la chiave segreta **sk** e un messaggio m e produce la firma (o tag) $\sigma = \{0,1\}^* \bigcup \{\bot\}$
- $\mathcal{VF}$: funzione deterministica che prende in input la chiave pubblica **pk**, un messaggio m ed una firma e restituisce un bit $0, 1$ a seconda che la firma per quel messaggio sia valida o meno

### Definizione di sicurezza (uf-cma)

Per definire formalmente la sicurezza di uno schema di firma digitale ci si può rifare alla definizione di **uf-cma** per **MAC**, facendo le dovute modifiche.
Una differenza importante, ad essempio, è che A non ha più bisogno dell'oracolo di verifica: gli basta infatti conoscere la chiave pubblica per essere in grado di verificare una coppia messaggio - tag in autonomia. 
A ha comunque a disposizione un oracolo che produce la firma di un messaggio e il suo obiettivo è quello di restituire una coppia messaggio - tag su un messaggio su cui non ha precedentemente interrogato l'oracolo, cioè è in grado di produrre un *falso*.

```pseudocode
ESP_{DS}^{uf-cma}(A):
	(pk, sk) <- KeyGen()
	(m, tag) <- A^{Sign(sk, .)}(pk)
	if VF(pk, m, tag) == 1 and m in messages and m was not aksed to the oracle before:
		return 1
	return 0
```

Il vantaggio in senso **uf-cma** di A è così definito:
$$
Adv_{\mathcal{DF}}^{uf-cma}(A) = Pr[Esp_{\mathcal{DS}}^{uf-cma}(A) = 1]
$$
Lo schema di firma digitale $\mathcal{DF}$ è sicuro in senso **uf-cma** se il vantaggio così definito è prossimo a 0 per ogni avversario A computazionalmente limitato.

## Firme RSA

Lo schema **RSA** è molto utilizzata anche nella generazione di firme digitali, con ben pochi cambiamenti legati alle funzioni che lo caratterizzano, elencate di seguito:

- $\mathcal{KeyGen}$: rimane praticamente invariato
  - Produce $N = pq$, prodotto di due numeri primi grandi non resi noti ed una coppia di esponenti $e, d \in Z_{\phi(N)}^*: ed \equiv 1 \mod \phi(N)$, il primo reso pubblico ed il secondo mantenuto segreto
- $\mathcal{Enc}$: l’algoritmo di cifratura diventa l’algoritmo di verifica
  - Lo spazio dei messaggi $M = Z_N^*$
- $\mathcal{Dec}$: l’algoritmo di decifratura diventa l’algoritmo di firma

```pseudocode
SIGN_{N,d}(m):
	if m not in M: /* m deve appartenere allo spazio dei messaggi */
		return "error"
	s <- m**d % N
	return s

VF_{N,e}(m, s)
	if m not in M or s not in M: /* m e il tag s devono appartenere allo spazio dei messaggi */
		return "error"
	if s**e = m % N:
		return 1
	return 0
```

Nonostante **RSA** continui ad essere considerato uno schema sicuro, la soluzione proposta è insicura. L'obiettivo dell'avversario, infatti, è quello di produrre un falso, cioè trarre in inganno la funzione di verifica. Esistono diversi avversari piuttosto semplici, come quelli che seguono, con un vantaggio in senso **uf-cma** molto alto.

```pseudocode
A(N, e):
	return (1, 1)
	
B(N, e):
	x <-R- M
	m <- x**e % N
	return (m, x)

C(N, e):
	m <-R- M
	m1 <-R- M - {1, m} /* spazio dei messaggi esclusi 1 ed m */
	m2 <- m * m1**(-1) % N
	x1 <- Sign(m1)
	x2 <- Sign(m2)
	x <- x1 * x2 % N
	return (m, x)
```

## Paradigma Hash-Inverti

Al fine di ovviare al grave problema di sicurezza, ma anche trovare delle condizioni di utilizzo più comode, come ad esempio l'allargamento dello spazio dei messaggi a elementi che non appartengano ad un gruppo, è stato introdotto il paradigma **Hash-Inverti**.

Prima di applicare la funzione già vista in precedenza, si pre-processa il messaggio utilizzando una funzione hash $H_N: \{0, 1\}^* \rightarrow Z_N^*$. Quindi i due algoritmi diventano 

```pseudocode
SIGN_{N,d}(m):
	y <- H(m)
	x <- y**d % N
	return x

VF_{N,e}(m, s)
	y <- H(m)
	y` <- x**e % N
	if y` == y:
		return 1
	return 0
```

La modifica sembra aver eliminato tutte quelle caratteristiche che ci creavano problemi nella prima versione. Tuttavia, introdurre un'altra funzione come quella hash porta a domandarci se ciò non apre la porta ad altre debolezze. Ad esempio, dobbiamo essere sicuri che la funzione che utilizziamo sia sicura in senso **cr2-kk**. Piuttosto che elencare tutte le condizioni necessarie, viene utilizzato il seguente esperimento per definirne una sufficiente.

```pseudocode
ESP_{DS}^{uf-cma}(A):
	((N, e), (p, q, d)) <- KeyGen()
	(m, s) <- A^{H(.), Sign(.)}(N, e)
	if VF(pk, m, s) == 1 and m in M and m was not aksed to the oracle before:
		return 1
	return 0
```

Il vantaggio in senso **uf-cma** di A è così definito:
$$
Adv_{\mathcal{DF}}^{uf-cma}(A) = Pr[Esp_{\mathcal{DS}}^{uf-cma}(A) = 1]
$$
Lo schema di firma digitale $\mathcal{DF}$ è sicuro in senso **uf-cma** se il vantaggio così definito è prossimo a 0 per ogni avversario A computazionalmente limitato.

### Dipendenza da **RSA**

Sia $\mathcal{DF}$ uno schema di firma digitale, e sia A un avversario che fa al più $q_s$ richieste di firma e $q_h$ richieste all'oracolo della funzione hash.
$$
\exists B \text{ che usa risorse comparabili ad } A \text{ che attacca RSA} : \\
Adv_{\mathcal{DF}}^{uf-cma}(A) \le q_h Adm_{\mathcal{RSA}}^{ow-rsa}(B)
$$
In altre parole, riuscire a rompere questo cifrario nell'ipotesi del random oracle permetterebbe di costruire un avversario in grado di risolvere il problema dell'invertibilità di **RSA**.

## BLS

Supponiamo di avere una funzione hash (che ha il ruolo di random oracle) $H: \{0, 1\}^* \rightarrow G_1$. SI considerino due elementi $g, h \in G_2 \quad h = g^x$. La chiave segreta è $x$.

```pseudocode
SIGN(x, m):
	y <- H(m)
	s <- y**x
	return s

VF(pk, s, m):
	y <- H(m)
	if e(y, h) == e(s, g)
		return 1
	return 0
```

Dato che i gruppi considerati sono ciclici, si può considerare che $(g, y, h, \sigma) = (g, \hat{g}^a, g^x, \hat{g}^{ax})$, con $g$ generatore di $G_2$ e $\hat{g}$ generatore di $G_1$.
Applicando la mappa bilineare abbiamo che $e(y, h) = e(\hat{g}^a, g^x) = e(\hat{g}, g)^{ax}$. Lo stesso è vero per la seconda coppia. Infatti $e(\sigma, g) = e(y^x, g) = e(\hat{g}^{ax}, g) = e(\hat{g}, g)^{ax}$. Quindi $e(y, h) = e(\sigma, g)$.


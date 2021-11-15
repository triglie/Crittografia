<h2 style="text-align:center">Prova in Itinere del 15 Novembre 2017</h2>

#### 1) Definire formalmente il concetto di perfetta sicurezza

Un cifrario $SE=(\mathcal{KeyGen}, \mathcal{ENC}, \mathcal{DEC})$ si dice perfettamente sicuro se verifica la seguente proprietà. 
Considerando gli gli insiemi $\mathcal{M, C, K}$ come, rispettivamente, spazio dei messaggi, dei crittotesti e delle chiavi
$$
\forall m_1, m_2 \in \mathcal{M}, \forall c \in \mathcal{C} \\
Pr[\mathcal{ENC}_k(m_1) = c] = Pr[\mathcal{ENC}_k(m_2) = c]
$$
Il altre parole, la probabilità che un qualunque messaggio produca un qualunque crittotesto è la stessa per qualsiasi messaggio e crittotesto validi considerati.

#### 2) Sia  $\mathcal{SE} = (\mathcal{KeyGen}, \mathcal{ENC}, \mathcal{DEC})$ un cifrario simmetrico e siano $M, K, C$ gli insiemi dei messaggi, delle chiavi e dei crittotesti, rispettivamente. Si considerino adesso i seguenti insiemi $M = \{1, 2, 3, 4, 5\}$ e $K = \{1, 2, 3, 4, 5, 6, 7, 8, 9, 10\}$ . Supponiamo di voler cifrare un solo messaggio $m \in M$, utilizzando una chiave (random) $k \in K$, come segue  $c = (k + 3m) \mod{11}$. E’ tale sistema sicuro in senso perfetto? Giustificare la risposta fornita.

**Non** si tratta di un cifrario simmetrico sicuro in senso perfetto poiché, per come è costruito, il risultato della cifratura di un messaggio può assumere 10 degli 11 possibili valori in $C$. Ad esempio, $m_1 = 1$ può assumere i valori $\mathcal{ENC}(m_1) \in \{4, 5, 6, 7, 8, 9, 10, 0, 1, 2\}$, mentre $m_2 = 4$ assumerà i valori  $\mathcal{ENC}(m_2) = \{2, 3, 4, 5, 6, 7, 8, 9, 10, 0\}$. Si può osservare che la probabilità che $Pr[\mathcal{ENC}_k(m_1) = 3] \ne Pr[\mathcal{ENC}_k(m_2) = 3] \rightarrow 0 \ne \frac{1}{10}$ 

#### 3) In classe, parlando del cifrario a blocchi AES, abbiamo discusso il campo di Galois GF($2^8$). Abbiamo visto che, in tale insieme, ogni byte può essere rappresentato come un polinomio di grado (al più) 7. Ricordando che m(x) = $(i) \qquad x^8 + x^4 + x^3 + x + 1$ è il polinomio irriducibile discusso a lezione, si calcoli la somma ed il prodotto dei seguenti due byte: $(1)\quad x^7 + x^5 + x^3 + 1 \qquad (2)\quad x^6 + x^3 + x^2 + x$

$$
(1) + (2) = x^7+x^6+x^5+x^4+x^2+1 \\
(1) * (2) = x^{13} + x^{10} + x^9 + x^8 + x^{11} + x^8 + x^7 + x^6 + x^9 + x^6 + x^5 + x^4 + x^6 + x^3 + x^2 + x / (i)= \\ 
= x^{13}+x^{11}+x^{10}+x^7+x^6+x^5+x^4+x^3+x^2+x / (i)\\
= x^{11}+x^{10}+x^9+x^8+x^7+x^4+x^3+x^2+x / (i) \\
= x^{10}+x^9+x^8+x^6+x^2+x / (i) \\
= x^9+x^8+x^5+x^3+x / (i) \\
= x^8+x^4+x^3+x^2 / (i) \\
= x^2+x+1
$$

####  4) Definire formalmente il concetto di funzione pseudocasuale

Una funzione si definisce pseudocasuale se il suo comportamento è indistinguibile da un utilizzatore che vi ha accesso in un modello blackbox, cioè ha solo la possibilità di conoscere l'output della funzione su un input arbitrario valido. Per definire formalmente il concetto di indistinguibilità si utilizza il seguente esperimento:

```pseudocode
ESP_{G_k}^prf-1:
	k <-R {0, 1}^l
	return A^{G_k}

ESP_{G_k}^prf-0:
	f <-R F(D, R)
	return A^{f}
```

Si definisce il vantaggio dell'avversario A come $ Adv(a) = | Pr[ESP_{G_k}^{prf-1}(A)=1] - Pr[ESP_{G_k}^{prf-0}(A)=1] |$.
Una prf è definita indistinguibile se questo valore è prossimo a 0.

#### 5) Sia $F : \{0, 1\}^k \times \{0, 1\}^{l+1} \rightarrow \{0, 1\}^L$ una funzione pseudocasuale sicura. Vogliamo utilizzare F per costruire una funzione $G : \{0, 1\}^k \times \{0, 1\}^{2l} \rightarrow \{0, 1\}^{2L}$ , nel seguente modo (il simbolo || denota l’operazione di concatenazione): 

```pseudocode
G_k(x):
	x1 || x2 <- x /* |x1| = |x2| = l */
	y1 <- F_k(x1||0)||F_k(x2||1)
	y2 <- F_k(¬x1||0)||F_k(¬x2||1)
	y <- y1 ⊕ y2
	return y
```

#### Dimostrare formalmente che G non è una funzione pseudo-casuale sicura. 

Per dimostrare il fatto che $G_k$ non sia una funzione pseudo-casuale sicura deve essere possibile costruire un attacco che permetta ad un avversario con potenza di calcolo limitata di avere un vantaggio non trascurabile. Immaginiamo il seguente attacco:

```pseudocode
A(G_k):
	x <-R {0, 1}^2l
	y1 <- O_{G_k}(x)
	y2 <- O_{G_k}(¬x)
	if y1 = y2: return 1
	else: return 0
```

Grazie a questa costruzione, sappiamo che la funzione $G_k$ produrrà gli stessi $y_1, y_2$, solo invertendoli. Dato che però lo ⊕ è commutativo, il risultato finale sarà lo stesso. Vediamo quindi il vantaggio di A.

- **L'oracolo usa $G_k$:** la stringa possiede la caratteristica cercata ed A indovinerà sicuramente.
- **L'oracolo usa una funzione casuale:** la probabilità di essere tratti in inganno è $\frac{1}{2^l}$, che rappresenta la probabilità che l'output della funzione abbia, per coincidenza, la caratteristica che cerchiamo.

$$
Adv(A) = |Pr[ESP_{G_k}^{prf-1}(A) = 1] - Pr[ESP_{G_k}^{prf-0}(A) = 1]| = 1 - \frac{1}{2^{2L}} \gg 0
$$

La funzione $G_k$ **non** è una funzione pseudo-casuale sicura.

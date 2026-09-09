# Chapitre 3 -- SystemConfigGlobal : enregistrer une enclave comme signataire de confiance

`contracts/src/SystemConfigGlobal.sol` herite de `NitroValidator` (une
bibliotheque externe qui sait verifier cryptographiquement un document
d attestation AWS Nitro) et maintient deux mappings cles :
`validPCR0s` (les mesures de code d enclave approuvees par le
proprietaire) et `validSigners` (les adresses dont la cle a ete prouvee
provenir d une enclave avec un PCR0 valide).

`registerPCR0`, reservee au owner, marque une mesure de code comme
digne de confiance -- c est la decision humaine et administrative du
systeme : l operateur de la chaine a inspecte et approuve le binaire
exact qui tournera dans l enclave. `registerSigner`, elle, est
automatisee et cryptographique : elle prend un document d attestation
brut (`attestationTbs`) et sa signature, appelle
`validateAttestation` (fournie par `NitroValidator`) pour verifier la
chaine de certificats X.509 remontant a la racine AWS Nitro, extrait le
PCR0 mesure dans le document et exige qu il figure dans `validPCR0s`,
verifie que l attestation n a pas plus de `MAX_AGE` (60 minutes)
-- une protection contre le rejeu d une vieille attestation --, puis
extrait la cle publique embarquee dans l attestation (au format ANSI
X9.62, le prefixe `0x04` etant ignore), la hache, et derive l adresse
Ethereum correspondante pour la marquer valide dans `validSigners`.

Le lien avec le chapitre 2 est direct : c est cette meme table
`validSigners`, alimentee uniquement par ce processus d attestation
verifiable onchain, que `OutputOracle.proposeL2Output` consulte pour
decider si une signature d output est acceptable. Toute la confiance du
systeme repose donc sur deux maillons : la decision administrative
(quel code merite d etre execute dans l enclave, via `registerPCR0`) et
la preuve cryptographique fournie par AWS Nitro lui-meme (que ce code
precis tourne bien et a genere cette cle, via `registerSigner`).

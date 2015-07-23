# Search

1. building of search index happens locally on each peer
2. each peer publishes a DAG-serialized, likely simplified version of its local search index
2. discovery of newly indexable content happens via DHT find-providers query for deterministically calculated content IDs

## Index in Pubtree

- indexing peers publish a tree of search tokens/facets with each leaf being a "list" node which links to track metadata objects which define content matching that search token
- something like this:
  ```yaml
  pubtree:
    index:
      QmY4Rs1sWinJgeNmME7VpAZWooUouNHF9TSWJ4oWVdhJ41: # hash of 'artist:Tame Impala'
        - QmTLZn8wTHzQ4Vh9vxi68uN4HiR7EB26ro6paNMP8k61GQ
        - QmTi5wvUuSnzs1joycSfoowEviytE7vAcrsciruHhTmuDq
        - QmVZahjQy8fgkxD6fKNKuuY2uBohbkf2JnZ6syhpkrWsSo
        - QmaHXADi79HCmmGPYMmdqvyemChRmZPVGyEQYmo6oS2C3a
      QmcpRKGaSeby5XKmA5dc2zTGwSiAe7JVyRGL2jCi873Aij: # hash of 'token:tame'
        - QmTLZn8wTHzQ4Vh9vxi68uN4HiR7EB26ro6paNMP8k61GQ
        - QmTi5wvUuSnzs1joycSfoowEviytE7vAcrsciruHhTmuDq
        - QmVZahjQy8fgkxD6fKNKuuY2uBohbkf2JnZ6syhpkrWsSo
        - QmaHXADi79HCmmGPYMmdqvyemChRmZPVGyEQYmo6oS2C3a
        - QmTeCatDYBWSpSoqV3HFsrdS7U3SpmojywFnP6tt5irRqK
        - Qmet7SbXQGwuCueLukMgk6tdZkeoKqNGr4yvHpF1CkuDyU
      QmZisKuoCZfDuRf3nkt1Xm7LXoScL31RWNGuAMeMTeogJe: # hash of 'token:love'
        - QmSzY5u4qtmfU8DR24SQzF5f6JNnzmxLoqLVnmbou56aAn
        - QmTeCatDYBWSpSoqV3HFsrdS7U3SpmojywFnP6tt5irRqK
        - QmYJPtDLaCqKUDBYUKWhKvPyckgWubXPKoAH3kth351YcW
        - Qmet7SbXQGwuCueLukMgk6tdZkeoKqNGr4yvHpF1CkuDyU
        - QmVq6eUjrUHckcUEbJF6pSdfWAAsFwi57jtdm5Sn5VFvRp
        - QmSdveHM1ShmHEjrNyLdkY9MYPyEKCgVvWXHvvbXCTP951
        - QmXJS1PUTdmRcxmDawUCBV6QdHV6NMxzDoFD44TULssSZE
        - Qmdq7c5kpLfsWi4aQpDM2T7g6pUz48iSdmKK6XyfZ9HkUr
      QmbuJEJBywtV9S9gTK7d21FkFhvAnV5J6iqD4siKZApFeC: # hash of 'genre:blues'
        - QmWSXsLMqSKBT4msxvkNrAivELvfvXmE3LQ9xzSoNBAyuz
        - QmTx78WMfVC5XSirzoheZYUL7yJe4WXW2XjvPDqVKy1JLX
        - QmbQm1W47hgHvo2Ytj1DNKVymc6KsXvPYW5AQxvi85LcGr
        - QmahXc6ZJdMUtu3xUrwveSYC5VqHHeBqTLzqdtops1LjaG
        - QmUmsfaqNtWpGXuYGZ6FeC2pSLvP6qhVoYvYAmo9J4YvtZ
        - QmVGpQGZWCJLVREtuwsUfcpt7y4bnH3tpaQQ4gHiRyLPXL
    peers:
      - QmVaPTddRyjLjMoZnYufWc5M5CjyGNPmFEpp5HtPKEqZFG
      - QmeSJ1cekSdxgzmJDnH7nyJu3HCjsU4nKCxNkh5wABE9HN
      - ...
    contents:
      - Qmdx97coS2FKH2Wc1sr3fUnrw3ooicyfBcaEKQx9vdiGU2
      - QmWV2B5h6esbJZXyksPyb9bxrHYzkeW8HVYit8vJbkk6Pz
      - ...
    release: QmUPWWPbGb8rpr8R9MsA8SWYuT39ADZfdNCKehoGa1BNGH
  ```

  We can use some digest of the `artist:XXX`-style label in order to step around possible encoding or length issues for certain values (SHA2-256 Base58-encoded multihash used here). Additionally, it's this hash which will have been used for discovery.


## Example flow

### Givens:
  - Bob doesn't yet know about peer Alice or her collection.
  - Alice has indexed and published her index including some Tame Impala tracks.

### Index Process

1. Alice's client becomes aware of a track metadata node ID, `QmUxML1ifHvnWtxz1cqWGdR4JGsj2QdHavQqkPGFDn41Pq`, which represents:
    ```yaml
    artist: Tame Impala
    title: Let It Happen
    album: Currents
    year: 2015
    ```

2. The track is given to the local search indexing process to be tokenized and ingested.

3. Following ingestion by the search engine locally, Alice's peer can being publishing its knowledge of the track. Regardless the style and manner in which it is indexed locally, it's represented in the pubtree in using these discovery tokens:

    ```yaml
    - 'album:Currents'
    - 'artist:Tame Impala'
    - 'title:Let It Happen'
    - 'token:2015'
    - 'token:currents'
    - # 'token:it' -- stopwords like 'it' are excluded from tokenized output
    - 'token:happen'
    - 'token:impala'
    - 'token:let'
    - 'token:tame'
    - 'year:2015'
    ```

4. Each are now actually added to IPFS as Clubnet Badges with no time-stamp component.

5. The digest of each token is used to install the track into the pubtree index tree under each token:

    ```yaml
    - QmZghvsL7yh4LyjAiAsSbCevL3G51zCk558PpXR12vKLQ5 # album:Currents
    - QmY4Rs1sWinJgeNmME7VpAZWooUouNHF9TSWJ4oWVdhJ41 # artist:Tame Impala
    - QmX7nojZtdsCqb68FhYw9x1GhJr3jNkDugNo1iuhjG5fJf # title:Let It Happen
    - QmbTSPe7SjbGYCbWEKpzHXpJjGoyUrqvVgUkmHzQ6pEukf # token:2015
    - QmTBCgdQ8qTccgkAFiQDPQWW8AhwT1WTLV19M25PfsxgUn # token:currents
    - QmNhFozGNBqjaZEsAf2gajRBd81XrA8y7ARTQncHgDj7AC # token:happen
    - QmQtUnVh6NjA1esQbJF2E3hREFxVN4vvrx8KvwEo7Cx1RH # token:impala
    - QmXJJpzyhPvgJvXrgx1JPq8AeaTvW14XtujDRYakzDcgSx # token:let
    - QmcpRKGaSeby5XKmA5dc2zTGwSiAe7JVyRGL2jCi873Aij # token:tame
    - Qmf4JVpWAUfEF4nze4Ag4hkRUyLwWCTA1spnbNG37urobf # year:2015
    ```

6. Alice's client publishes the latest version of her pubtree including these tokens, each with the new track listed under it.

    ```yaml
    pubtree:
      peers:
        - ...
      contents:
        - ...
        - QmUxML1ifHvnWtxz1cqWGdR4JGsj2QdHavQqkPGFDn41Pq
      index:
        QmZghvsL7yh4LyjAiAsSbCevL3G51zCk558PpXR12vKLQ5: # album:Currents
          - QmUxML1ifHvnWtxz1cqWGdR4JGsj2QdHavQqkPGFDn41Pq
          - ...
        QmY4Rs1sWinJgeNmME7VpAZWooUouNHF9TSWJ4oWVdhJ41: # artist:Tame Impala
          - QmUxML1ifHvnWtxz1cqWGdR4JGsj2QdHavQqkPGFDn41Pq
          - ...
        QmX7nojZtdsCqb68FhYw9x1GhJr3jNkDugNo1iuhjG5fJf: # title:Let It Happen
          - QmUxML1ifHvnWtxz1cqWGdR4JGsj2QdHavQqkPGFDn41Pq
          - ...
        QmbTSPe7SjbGYCbWEKpzHXpJjGoyUrqvVgUkmHzQ6pEukf: # token:2015
          - QmUxML1ifHvnWtxz1cqWGdR4JGsj2QdHavQqkPGFDn41Pq
          - ...
        QmTBCgdQ8qTccgkAFiQDPQWW8AhwT1WTLV19M25PfsxgUn: # token:currents
          - QmUxML1ifHvnWtxz1cqWGdR4JGsj2QdHavQqkPGFDn41Pq
          - ...
        QmNhFozGNBqjaZEsAf2gajRBd81XrA8y7ARTQncHgDj7AC: # token:happen
          - QmUxML1ifHvnWtxz1cqWGdR4JGsj2QdHavQqkPGFDn41Pq
          - ...
        QmQtUnVh6NjA1esQbJF2E3hREFxVN4vvrx8KvwEo7Cx1RH: # token:impala
          - QmUxML1ifHvnWtxz1cqWGdR4JGsj2QdHavQqkPGFDn41Pq
          - ...
        QmXJJpzyhPvgJvXrgx1JPq8AeaTvW14XtujDRYakzDcgSx: # token:let
          - QmUxML1ifHvnWtxz1cqWGdR4JGsj2QdHavQqkPGFDn41Pq
          - ...
        QmcpRKGaSeby5XKmA5dc2zTGwSiAe7JVyRGL2jCi873Aij: # token:tame
          - QmUxML1ifHvnWtxz1cqWGdR4JGsj2QdHavQqkPGFDn41Pq
          - ...
        Qmf4JVpWAUfEF4nze4Ag4hkRUyLwWCTA1spnbNG37urobf: # year:2015
          - QmUxML1ifHvnWtxz1cqWGdR4JGsj2QdHavQqkPGFDn41Pq
          - ...
    ```

### Search Process

1. Bob searches for "Tame Impala" in UI

2. Bob's client performs the search against the local search engine and immediately displays the best results it has locally. The UI indicates searching is ongoing. To find more content which matches the search, the client starts a new content discovery routine specific to the search string supplied:

2. Bob's client tokenizes and normalizes the search string:
    ```yaml
    - tame
    - impala
    ```

3. Because it's performing a tokenized text query, it appends each token from the query to `token:` to produce the peer-discovery clubnet tokens:
    ```yaml
    - 'token:tame'
    - 'token:impala'
    ```

4. As typical with Clubnet, each discovery token is hashed:
    ```yaml
    - QmcpRKGaSeby5XKmA5dc2zTGwSiAe7JVyRGL2jCi873Aij
    - QmQtUnVh6NjA1esQbJF2E3hREFxVN4vvrx8KvwEo7Cx1RH
    ```

5. We now have content IDs for which we can query the DHT for providers.
    ```yaml
    - ipfs dht findprovs QmcpRKGaSeby5XKmA5dc2zTGwSiAe7JVyRGL2jCi873Aij
    - ipfs dht findprovs QmQtUnVh6NjA1esQbJF2E3hREFxVN4vvrx8KvwEo7Cx1RH
    ```

6. Because Alice has indexed some tracks with some value of the metadata including the token `tame`, the first of these queries returns Alice's peer ID, likely among others.
    ```yaml
    - QmRuqS9hv17XT752DBKinP5YxqKSVVtmv9huQEKsch75qR
    ```

7. Alice's knowledge of tracks matching the token `tame` can be retrieved using the following path:

    ```yaml
    /ipns/<peer-id>/index/<hash-of-discovery-token>
    /ipns/QmRuqS9hv17XT752DBKinP5YxqKSVVtmv9huQEKsch75qR/index/QmcpRKGaSeby5XKmA5dc2zTGwSiAe7JVyRGL2jCi873Aij
    ```

    Additionally, to aide in standard peer-discovery processes, Alice's Peer ID is added to Bob's client's internal peer list.

    > At this point, we could also perform the same request for every other peer which Bob already knows about (assuming that either Bob has knowingly not fetched and indexed all of the content of all of the peers he knows about, or at least that those peers are likely to have newly discovered content which Bob doesn't yet know about)

8. The object retrieved at that path will be a 'list' of metadata-nodes each of which point to content which matches this search token.

9. Each of these are retrieved and then indexed locally by Bob's client's local search indexing process.

10. As changes to the local index are made, additional search results appear in Bob's UI. Results are ordered based on their search rank given the full query string and how well each result matches (this behavior is up to the client to define).

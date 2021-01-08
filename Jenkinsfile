pipeline {
    
   agent any
   
    triggers {
        cron('H 0 * * 3')
    }
    
    environment{
        ENCODE = "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
    }
    
   stages {
      stage('Backing Up Vaults') {
         steps {
            sh '''
TSTAMP=`date +%Y%m%d%H%M`
eval $(cat /home/ubuntu/op-backup/some.txt | op signin 'username')

vaults=$(op list items | jq '.[]' | jq --raw-output '.vaultUuid' | uniq)
key=""
for vault in $vaults
do
    echo "  - getting vault ${vault}"
    items=$(op list items --vault ${vault} | jq '.[]' | jq --raw-output '.uuid')
    document=""
    for item in $items
        do
            echo "  - getting item ${item} in vault ${vault}"
            key=$(op get item ${item}),$key
                documentId=$(op get item ${item} | jq '.details' | jq '.documentAttributes' | jq --raw-output '.documentId')
                vaultname=$(op get vault ${vault} | jq --raw-output '.name')
                case $documentId in
                    null)
                        continue
                        ;;
                    *)
                        echo "  - getting document $(op get item ${item} | jq '.details' | jq '.documentAttributes' | jq --raw-output '.documentId')"
                        document=$(op get item ${item} | jq '.overview' | jq --raw-output '.title'):$(op get document ${item}):$(op get item ${item} | jq '.overview' | jq --raw-output '.title')+$document
                        ;;
                esac
        done
    if ! [ -z "$document" ]
        then
            echo $document | openssl aes-256-cbc -pbkdf2 -a -out $vaultname.documents-encrypt.$TSTAMP.txt -k $ENCODE
    fi
done

echo $key | openssl aes-256-cbc -pbkdf2 -a -out pass-encrypt.$TSTAMP.json -k $ENCODE


aws s3 cp . 'destination-bucket' --recursive

find /home/ubuntu/op-backup/backup/ -type f -mtime +190 -delete
            
cp * /home/ubuntu/op-backup/backup/
            '''
         }
      }
   }
   
  post { 
        always { 
            cleanWs()
        }
    }
    
}

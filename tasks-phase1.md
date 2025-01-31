IMPORTANT ❗ ❗ ❗ Please remember to destroy all the resources after each work session. You can recreate infrastructure by creating new PR and merging it to master.
  
![img.png](doc/figures/destroy.png)

1. Authors: ✅

   Z2

   https://github.com/mik00laj/tbd-workshop-1
   
2. Follow all steps in README.md. ✅
   
Zrealizowaliśmy kroki 1–9 opisane w pliku README.md, czego efektem było wydanie pierwszej wersji (release). 
![PR](https://github.com/user-attachments/assets/75d06944-a74b-42df-bec0-8655f64d7ef4)
![PR2](https://github.com/user-attachments/assets/1da0201f-8b0b-4a09-bbec-c7e002a60626)


3. Select your project and set budget alerts on 5%, 25%, 50%, 80% of 50$ (in cloud console -> billing -> budget & alerts -> create buget; unclick discounts and promotions&others while creating budget). ✅

![img.png](doc/figures/discounts.png)
![Budget](https://github.com/user-attachments/assets/34103e76-2a54-448c-bf1c-a78d7a661bc0)
![Budget2](https://github.com/user-attachments/assets/3d2ff17f-48e6-437a-98ea-a595eb613def)

4. From avaialble Github Actions select and run destroy on main branch. ✅
   
   Kończąc pierwszą udaną sesję pracy, poprzez GA dokonaliśmy zniszczenia aktualnej infrastruktury.
   ![Destroy](https://github.com/user-attachments/assets/a4a7375e-d0c2-4679-a9c3-389fe96e98b3)

5. Create new git branch and: ✅
    1. Modify tasks-phase1.md file. 
    
    2. Create PR from this branch to **YOUR** master and merge it to make new release.
   
   Rozpoczynając kolejną sesję pracy, zmergowaliśmy gałąź ze zmodyfikowanym plikiem tasks-phase1.md, by ponownie postawić infrastrukturę. Kolejny release przeszedł pomyślnie.
![PR3](https://github.com/user-attachments/assets/d8df91dc-dc17-498f-9c93-0a1da0d4ce5a)
![PR4](https://github.com/user-attachments/assets/cddb19ef-3bc4-4201-90eb-3845385264da)


6. Analyze terraform code. Play with terraform plan, terraform graph to investigate different modules.✅

W katalogu modules znajdują się moduły Terraform,  są to między innymi: composer, data-pipeline, dataproc, dbt_docker_image, gcr, jupyter_docker_image, metastore, vertex-ai-workbench oraz vpc. W szczególności **przeanalizowaliśmy moduł vertex-ai-workbench**, a poniżej przedstawiono wynik wywołania komendy terraform graph.

![Terraform plan graph](https://github.com/user-attachments/assets/1197fc1e-2945-47ca-b02d-115318b20206)
![Terraform plan](https://github.com/user-attachments/assets/4b690f36-8017-41cb-8711-21f58399cd12)

Moduł `vertex-ai-workbench` umożliwia uruchomienie środowiska notebookowego opartego na Google Vertex AI, z obsługą PySpark i konektora GCS. Jest to przydatne w zadaniach związanych z analizą big data oraz uczeniem maszynowym. Moduł ten został skonfigurowany do automatycznego przypisywania odpowiednich ról IAM, zapewniania dostępu do zasobów w Google Cloud oraz uruchamiania kontenera PySpark z wymaganymi ustawieniami po starcie notebooka.

Moduł `vertex-ai-workbench` korzysta z następujących zasobów Google Cloud:

Instancja notebooka Vertex AI
- **`google_notebooks_instance.tbd_notebook`**: Zasób odpowiedzialny za uruchomienie instancji notebooka w środowisku Vertex AI. Jest to konfiguracja konkretnej instancji notebooka, która wykorzystuje zasoby `google_project_service.notebooks` oraz `google_storage_bucket_object.post-startup` do uruchamiania i początkowej konfiguracji.
- **`google_project_service.notebooks`**: Usługa umożliwiająca korzystanie z notebooków w ramach projektu Google Cloud.

Przechowywanie konfiguracji notebooka
- **`google_storage_bucket.notebook-conf-bucket`**: Bucket służący do przechowywania plików konfiguracyjnych notebooka.
- **`google_storage_bucket_object.post-startup`**: Obiekt w bucket, zawierający skrypt uruchamiany po starcie instancji notebooka, co pozwala na jej wstępną konfigurację. Skrypt jest częścią konfiguracji zawartej w `notebook-conf-bucket`.

Zarządzanie dostępem i uprawnieniami
- **`google_project_iam_binding.token_creator_role`**: Zasób przypisujący rolę w projekcie, która umożliwia generowanie tokenów.
- **`google_storage_bucket_iam_binding.binding`**: Zasób przypisujący role IAM (Identity and Access Management) do bucketu, zapewniając dostęp do `notebook-conf-bucket` i umożliwiając prawidłowe działanie instancji notebooka.


Skrypt `notebook_post_startup_script.sh` uruchamiany jest po starcie instancji notebooka i wykonuje następujące czynności:

-  Ustala szczegóły instancji kontenera, takie jak nazwa, ID, adres IP oraz hostname.
-  Ustawia zmienne środowiskowe dla PySparka (`PYSPARK_SUBMIT_ARGS`), konfiguruje parametry, takie jak liczba executorów Spark, ilość pamięci oraz konektor GCS.
-  Restartuje kontener i wystawia porty do interakcji z notebookiem (`8080`, `16384`, `16385`, `4040`).
- Instalacja środowiska kernel PySparka w notebooku, aby użytkownik mógł korzystać z PySparka w Vertex AI Workbench.
   
7. Reach YARN UI
   
  W celu połączenia się z YARN UI użyto następującej komendy, która pozwala na utworzenie tunelu przez IAP oraz przekierowanie portów aplikacji:
  ```
  > gcloud compute ssh tbd-cluster-m --project=tbd-2024z-2 --zone=europe-west1-d --tunnel-through-iap -- -L 8088:localhost:8088
  ```
  Niezbędne było również skonfigurowanie odpowiednich reguł firewall'a dla portów 8088 local i remote oraz zezwolenie na ruch HTTP i HTTPS, aby było możliwe wyświetlenie GUI YARN w przeglądarce na lokalnym hoście.

  Po wpisaniu w przeglądarkę internetową adresu: http://localhost:8088/cluster wyświetla się następujący widok:
  ![Yarn GUI](screenshots/yarn_gui.png)

8. Draw an architecture diagram (e.g. in draw.io) that includes:
    1. VPC topology with service assignment to subnets
    ![Topology](screenshots/topology.jpg)
    2. Description of the components of service accounts

        **tbd-2024z-2-data@tbd-2024z-2.iam.gserviceaccount.com:** To konto usługowe, znane jako "tbd-composer-sa", pełni rolę orkiestratora dla środowisk Cloud Composer, klastrów Dataproc oraz powiązanych zadań. Jest odpowiedzialne za zarządzanie i koordynację operacji związanych z danymi w obrębie środowiska.

        **tbd-2024z-2-lab@tbd-2024z-2.iam.gserviceaccount.com:** Określane jako "tbd-terraform", to konto usługowe jest wykorzystywane wyłącznie do działań związanych z Terraform. Umożliwia komunikację i zarządzanie infrastrukturą projektu w Google Cloud z poziomu Terraform, zapewniając płynną integrację i kontrolę zasobów.

        **727404351958-compute@developer.gserviceaccount.com:** To konto usługowe, określone jako "iac", wspiera integrację między GitHub a usługami Google Cloud. Odgrywa kluczową rolę w zarządzaniu tokenami dostępu i zapewnianiu płynnej komunikacji między tymi platformami.

        ![Service_accounts](screenshots/service_accounts.jpg)

    3. List of buckets for disposal

        **tbd-2024z-2-state** - Bucket, który przechowuje informacje o infrastrukturze i jej stanie, takie jak pliki Terraform.

        **tbd-2024z-2-data** - Bucket, który przechowuje rzeczywiste dane generowane przez aplikacje (surowe dane, wyniki aplikacji, logi).

        **tbd-2024z-2-conf** - Bucket, który przechowuje pliki konfiguracyjne i skrypty.

        **tbd-2024z-2-code** - Bucket, który przechowuje kod wykonywalny, kod źródłowy oraz biblioteki dla Apache Spark.

        ![Buckets](screenshots/buckets_.jpg)

    4. Description of network communication (ports, why it is necessary to specify the host for the driver) of Apache Spark running from Vertex AI Workbech

        0.10.10.2: tbd-cluster-w-1 - Worker

        10.10.10.3: tbd-cluster-w-0 - Worker

        10.10.10.4: tbd-cluster-m - Master
        
        10.10.10.5: tbd-2024-2-notebook - Maszyna wirtualna JupyterLab Notebook

        Vertex AI Workbench driver Apache Spark może działać na innej maszynie lub w odrębnym kontenerze niż pozostałe komponenty klastra Spark. Aby zapewnić poprawną komunikację, konieczne jest wskazanie konkretnego hosta dla drivera. Dzięki temu driver Spark wie, do którego klastra przesyłać zadania oraz skąd odbierać przetworzone dane. Określenie hosta umożliwia nawiązywanie stabilnych połączeń między driverem a workerami w klastrze, co jest niezbędne do sprawnego przeprowadzania operacji obliczeniowych.

        Port drivera: 30000 — wykorzystywany do zarządzania komunikacją między driverem a komponentami klastra.

        Block manager: 30001 — służy do przesyłania bloków danych między instancjami w klastrze, zapewniając płynne przekazywanie wyników.

        Port drivera: 30000
        Block manager: 30001
  

9. Create a new PR and add costs by entering the expected consumption into Infracost
For all the resources of type: `google_artifact_registry`, `google_storage_bucket`, `google_service_networking_connection`
create a sample usage profiles and add it to the Infracost task in CI/CD pipeline. Usage file [example](https://github.com/infracost/infracost/blob/master/infracost-usage-example.yml) 

   ```
    version: 0.2
    usage:
      google_artifact_registry.registry:
        storage_gb: 40

      google_storage_bucket.tbd_code_bucket:
        storage_gb: 80
        monthly_class_a_operations: 800
        monthly_class_b_operations: 400
        monthly_egress_data_gb: 40

      google_storage_bucket.tbd_data_bucket:
        storage_gb: 200
        monthly_class_a_operations: 1500
        monthly_class_b_operations: 800
        monthly_egress_data_transfer_gb:
          same_continent: 38  # Same continent.
          worldwide: 40     # Worldwide excluding Asia, Australia.
          asia: 5           # Asia excluding China, but including Hong Kong.
          china: 2            # China excluding Hong Kong.
          australia: 15 

      google_service_networking_connection.private_vpc_connection:
        monthly_data_processed_gb: 200
   ```

   ![Infracost Output](screenshots/infra_cost.png)
   ![Infracost Output2](screenshots/infra_cost2.png)

10. Create a BigQuery dataset and an external table using SQL
    
    Poniższe polecenie wykorzystano aby stworzyć BigQuery demo dataset:
    ```
    CREATE SCHEMA IF NOT EXISTS demo OPTIONS(location = 'europe-west1');

    CREATE OR REPLACE EXTERNAL TABLE demo.shakespeare
      OPTIONS (
      
      format = 'ORC',
      uris = ['gs://tbd-2024z-2-data/data/shakespeare/*.orc']);


    SELECT * FROM demo.shakespeare ORDER BY sum_word_count DESC LIMIT 5;
    ```

    Niestety, composer z jakiegoś powodu nie stworzył data-daga i przez to nie mieliśmy danych do realizacji tego ćwiczenia.
    
   
      ![BigQuery](screenshots/big_query.png)

    Dopiero po realizacji zadania z znalezieniem błędu w pyspark-job.py, możliwe było wygenerowanie demo dataset'u.

      ![BigQuery](screenshots/big_query_success.png)

    Format ORC nie wymaga zdefiniowanego z góry schematu tabeli, ponieważ stosuje podejście „schema-on-read”. Oznacza to, że schemat jest określany lub dedukowany dopiero podczas odczytu danych, a nie w momencie ich zapisu. Dzięki temu rozwiązanie jest bardziej elastyczne i szczególnie przydatne w sytuacjach, gdy schemat często się zmienia.

  
11. Start an interactive session from Vertex AI workbench:

    Podobnie jak przy nawiązywaniu połączenia SSH z YARN UI, użyliśmy prawie identycznej komendy do stworzenia tunelu przez IAP. Tym razem, jednak udało się nam to przy zmienieniu portu na 8080 i dodanie analogicznych reguł do firewall'a:

    ```
    > gcloud compute ssh tbd-2024z-2-notebook --project=tbd-2024z-2 --zone=europe-west1-b --tunnel-through-iap -- -L 8080:localhost:8080
    ```
    ![Jupyter notebook](screenshots/jupyter_notebook_success.png)

   
12. Find and correct the error in spark-job.py

    Błąd polegał na tym, że w skrypcie spark-job.py była zapisana nieprawidłowa nazwa Bucketa. Udało się nam go znaleźć analizując informacje z logów w konsoli Google.
    ```
      DATA_BUCKET = "gs://tbd-2025z-9900-data/data/shakespeare/"
    ```

    Po podmienieniu nazwy wygląda to następująco:
    ```
      DATA_BUCKET = "gs://tbd-2024z-2-data/data/shakespeare/"
    ```
    Dzięki temu, prawidłowo generuje się demo dataset 'shakespeare' dla zadania z punktu dotyczącego BigQuery.

13. Additional tasks using Terraform: ✅

    1. Add support for arbitrary machine types and worker nodes for a Dataproc cluster and JupyterLab instance

    Arbitraty machine types for a Dataproc cluster are already supported via the `machine_type` variable, so it was skipped.

    ['modules/dataproc/variables.tf'](modules/dataproc/variables.tf)

    ```
    variable "worker_count" {
      type        = number
      default     = 2
      description = "Number of worker nodes"
    }
    ```

    ['modules/dataproc/main.tf'](modules/dataproc/main.tf)

    ```
    cluster_config {
      ...
      worker_config {
        num_instances = var.worker_count
        ...
      }
    }
    ```

    ['modules/vertex-ai-workbench/variables.tf'](modules/vertex-ai-workbench/variables.tf)

    ```
    variable "machine_type" {
      type        = string
      default     = "e2-standard-2"
      description = "Machine type to use for the notebook instance"
    }
    ```

    ['modules/vertex-ai-workbench/main.tf'](modules/vertex-ai-workbench/main.tf)

    ```
    resource "google_notebooks_instance" "tbd_notebook" {
      ...
      machine_type = var.machine_type
      ...
    }
    ```
    
    2. Add support for preemptible/spot instances in a Dataproc cluster

    ['modules/dataproc/variables.tf'](modules/dataproc/variables.tf)

    ```
    variable "preeemptible_worker_count" {
      type        = number
      default     = 0
      description = "Number of preemptible worker nodes"
    }
    ```

    ['modules/dataproc/main.tf'](modules/dataproc/main.tf)

    ```
    preemptible_worker_config {
      num_instances = var.preeemptible_worker_count

      disk_config {
        boot_disk_type    = "pd-standard"
        boot_disk_size_gb = 100
      }

      preemptibility = "SPOT"
    }
    ```
    
    3. Perform additional hardening of Jupyterlab environment, i.e. disable sudo access and enable secure boot

    ['modules/vertex-ai-workbench/main.tf'](modules/vertex-ai-workbench/main.tf)

    ```
    metadata = {
      vmDnsSetting : "GlobalDefault"
      notebook-disable-root: true
    }
    ```

    ```
    shielded_instance_config {
      enable_secure_boot          = true
      enable_vtpm                 = true
      enable_integrity_monitoring = true
    }
    ```

    4. (Optional) Get access to Apache Spark WebUI

    ['modules/dataproc/main.tf'](modules/dataproc/main.tf)

    ```
    cluster_config {
      endpoint_config {
        enable_http_port_access = true
      }
      ...
    }
    ```

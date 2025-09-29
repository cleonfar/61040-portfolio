concept AnimalIdentity 
  purpose represent individual animals with persistent identifiers and core attributes  
  principle  
    a user registers animals to track them individually across their lifecycle;  
    assigns each animal a unique tag and records identifying details;  
    updates status to reflect key transitions such as sale, death, or transfer;  
    and uses this identity to link the animal to other concepts like herd membership, production metrics, and lifecycle events.  

  state  
    a set of Animals with  
      an owner User  
      an id tag String  
      a species String  
      a breed String  
      a sex Enum [male, female, unknown]  
      a status Enum [alive, sold, deceased, transferred]  
      a notes (Set of Strings)  
      an optional birthDate Date  
      an optional mother Animal  
      an optional father Animal  
      an optional set of offspring (Set of Animals)  

  actions  
    registerAnimal (owner: User, tag: String, species: String, breed: String, birthDate: Date, sex: Enum): (animal: Animal)  
      effects create a new animal with this owner and attributes, status set to alive  

    updateStatus (animal: Animal, status: Enum, notes: String)  
      requires animal exists  
      effects set the animal’s status to the new value and record optional notes  

    editDetails (animal: Animal, species: String, breed: String, birthDate: Date, sex: Enum)  
      requires animal exists  
      effects update the animal’s identifying attributes  

    viewAnimal (tag: Tag): (animal: Animal)  
      requires an animal with this tag exists  
      effects return the animal and its attributes  

    listAnimals (owner: User): (animals: Set<Animal>)  
      effects return all animals owned by this user  



concept HerdGrouping \[Animal\]  
  purpose: organize animals into dynamic groupings for operational and analytical purposes  
  principle:  
    a user creates herds to group animals based on location, purpose, or management strategy;  
    adds or removes animals from herds as conditions change;  
    merges herds when combining groups, or splits them to separate animals;  
    moves animals between herds to reflect real-world transitions;  
    and views herd composition and history to support planning and analysis.  

  state  
    a set of Herds with  
      an owner User  
      a name String  
      a location String  
      a set of Animals  


  actions
    createHerd (owner: User, name: String, location: String): (herd: Herd)
      effects create a new herd with this owner, name, location, and no members

    addAnimal (herd: Herd, animal: Animal)
      requires herd exists and animal exists
      effects add the animal to the herd and record an add event

    removeAnimal (herd: Herd, animal: Animal)
      requires herd exists and animal is a member
      effects remove the animal from the herd and record a remove event

    moveAnimal (source: Herd, target: Herd, animal: Animal)
      requires both herds exist and animal is a member of source
      effects remove the animal from source, add to target, and record a move event

    mergeHerds (source: Herd, target: Herd)
      requires both herds exist
      effects move all animals from source to target, record a merge event, and archive source and target herd.

    splitHerd (source: Herd, target: Herd, animals: Set<Animal>)
      requires both herds exist and all animals are members of source
      effects move specified animals from source to target and record a split event

    viewComposition (herd: Herd): (animals: Set<Animal>)
      requires herd exists
      effects return current members of the herd



concept: GrowthRecords \[Animal\]
  purpose: track growth and feeding data for animals raised for meat production
  principle:  
    a user records performance data such as weight and feed consumption for individual animals;
    may record these metrics together or separately, depending on available data;
    uses this data to monitor rate of gain and feed efficiency;


  state:  
    a set of animals with  
      a set of WeightRecords with  
        a date Date  
        a weight Number  
        an optional notes String  


  actions:  
    recordWeight (animal: Animal, date: Date, weight: Number, notes: String)  
      requires animal exists  
      effects create a new weight record for this animal    

    viewWeightHistory (animal: Animal): (records: Set<WeightRecord>)  
      requires animal exists  
      effects return all weight records for this animal  


concept ReproductiveRecords \[Animal\]
  purpose track reproductive outcomes and offspring survivability for breeding animals
  principle
    a user records birth events for mother animals, optionally linking fathers and offspring;
    later records weaning outcomes for those offspring when the data becomes available;
    and uses this data to evaluate reproductive performance and inform breeding decisions.

  state
    a set of BirthRecords with  
      a mother animal
      an optional father Animal  
      a birthDate Date  
      a set of offspring Animals  
      a countBorn Number  
        an optional notes String  

    a set of WeaningRecords with  
      a mother animal
      a birthRecord BirthRecord  
      a weaningDate Date  
      a countWeaned Number  
      an optional notes String  

  actions:  
    recordBirth (mother: Animal, father: Animal?, birthDate: Date, offspring: Set<Animal>, countBorn: Number, notes: String?)  
      requires: mother exists and all offspring exist  
      effects: create a new birth record and link offspring to mother and optional father  

    recordWeaning (birthRecord: BirthRecord, weaningDate: Date, countWeaned: Number, notes: String?)  
      requires: birthRecord exists and weaningDate is after birthDate  
      effects: create a new weaning record linked to the birth record  

    viewBirths (animal: Animal): (records: Set<BirthRecord>)  
      requires: animal exists  
      effects: return all birth records where this animal is the mother or father  

    viewWeaning (birthRecord: BirthRecord): (record: WeaningRecord?)
      requires: birthRecord exists  
      effects: return the weaning record associated with this birth, if any  


concept DataAnalysis [Animal, Herd]  
  purpose generate summaries, comparisons, and trends based on recorded animal and herd data  
  principle  
    a user selects an individual animal or herd to generate a report;  
    queries performance records such as weight and growth rate;  
    analyzes reproductive outcomes including offspring counts and survivability;  
    and views aggregated results to support operational decisions.  

  state  
    a set of GeneratedReports with  
      a reportType Enum [growth, reproduction, herd composition]  
      a dateGenerated Date  
      a target Animal or Herd
      a set of metrics Enum [weight, rateOfGain, offspringPerBirth, weaningRate]  
      a summary String  
      a set of results (key-value pairs or tabular data)  
      an optional exportLink String  

  actions  
    generateReport (user: User, reportType: Enum, filters: Set<Filter>, metrics: Set<Enum>): (report: GeneratedReport)  
      effects: produce a report based on the specified parameters and store the results  

    updateBirthRecord (birthRecord: BirthRecord, birthDate: Date?, offspring: Set<Animal>?, countBorn: Number?, notes: String?)  
      requires: birthRecord exists  
      effects: update any provided fields in the birth record and leave the rest unchanged  


    viewReport (report: GeneratedReport): (summary: String, results: GeneratedReport)  
      requires: report exists  
      effects: return the summary and results of the report  

    listReports (): (reports: Set<GeneratedReport>)  
      effects: return all generated reports  

    deleteReport (report: GeneratedReport)  
      requires: report exists and belongs to the user  
      effects: remove the report from the system  
    

sync recordBirth  
  when AnimalIdentity.RegisterAnimal(owner: User, tag: String, species: String, breed: String, birthDate: Date, sex: Enum, mother)  
  then ReproductiveRecords.recordBirth(mother: Animal, father: Animal?, birthDate: Date, offspring: Set<Animal>, countBorn: Number, notes: String?) or ReproductiveRecords.updateBirthRecord (mother: Animal?, father: Animal?, birthDate: Date, offspring: Set<Animal>, countBorn: Number, notes: String?)  

  Create a birth record if there is one or add the animal to the birth record if there is one  


sync viewGrowth  
  when DataAnalysis.generateReport (animal, reportType: growth)  
  then GrowthRecords.viewGrowth (animal)  



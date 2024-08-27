
# Documentation for Search, Filter, Sort and Pagination Implementation

This documentation provides a step-by-step guide for implementing search, sort, filter, and pagination functionality using Prisma. The functionality is wrapped in an asynchronous function, making it easy to handle HTTP requests and database queries efficiently.

## 1. Parsing Filters and Pagination Options

Extract filterable fields, pagination, and sorting options from the request query. You can use a utility function like `pick` to select specific fields.

```ts
const filters = pick(req.query, adminFilterableFields); // ['name','email','search','contactNumber']
const options = pick(req.query, ['limit', 'page', 'sortBy', 'sortOrder']);
```

Calculate the pagination data using a helper function like `paginationHelper`.

```ts
const { page, limit, skip } = paginationHelper.calculatePagination(options);
```

## 2. Building the Filter Conditions

Extract the search and filter data from the `filters` object. Use conditional logic to create the filter conditions dynamically.

```ts
const { search, ...filterData } = filters;
const andConditions: Prisma.AdminWhereInput[] = [];

// adminSearchAbleFields = // ['name','email','contactNumber']

if (search) {
    andConditions.push({
        OR: adminSearchAbleFields.map(field => ({
            [field]: {
                contains: search,
                mode: 'insensitive'
            }
        }))
    });
}

if (Object.keys(filterData).length > 0) {
    andConditions.push({
        AND: Object.keys(filterData).map(key => ({
            [key]: {
                equals: (filterData as any)[key]
            }
        }))
    });
}
```

## 3. Constructing the Prisma Query

Build the `where` clause using the filter conditions. Then, use the `prisma.admin.findMany` method to fetch the data. Apply sorting and pagination using the `orderBy`, `skip`, and `take` options.

```ts
const whereConditions: Prisma.AdminWhereInput = {
    AND: andConditions
};

const result = await prisma.admin.findMany({
    where: whereConditions,
    skip,
    take: limit,
    orderBy: options.sortBy && options.sortOrder
        ? { [options.sortBy.toString()]: options.sortOrder }
        : { createdAt: 'desc' }
});
```

## 4. Counting the Total Records

Use the `prisma.admin.count` method to get the total number of records that match the filter conditions.

```ts
const total = await prisma.admin.count({
    where: whereConditions
});
```

## Conclusion

This pattern allows for dynamic search, sorting, filtering, and pagination in your Prisma queries. Adjust the fields and logic to suit your specific needs.

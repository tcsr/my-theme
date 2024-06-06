This is a Custom keycloack theme

app.get('/einvoice/invoiceListPaginated', (req, res) => {
    const { page = 0, size = 10, search = '', filter = '' } = req.query;

    fs.readFile(invoiceListPaginated, 'utf8', (err, data) => {
        if (err) {
            console.error(err);
            return res.status(500).send('Error reading data');
        }

        const jsonData = JSON.parse(data);
        let content = jsonData.content;

        // Apply search filter
        if (search) {
            content = content.filter(invoice =>
                Object.values(invoice).some(value =>
                    value.toString().toLowerCase().includes(search.toLowerCase())
                )
            );
        }

        // Apply custom filter (example: filter by irbmResponse)
        if (filter) {
            content = content.filter(invoice =>
                invoice.irbmResponse.toLowerCase() === filter.toLowerCase()
            );
        }

        const totalElements = content.length;

        // Apply pagination
        const startIndex = page * size;
        const endIndex = startIndex + parseInt(size);
        const paginatedContent = content.slice(startIndex, endIndex);

        const response = {
            content: paginatedContent,
            pageable: {
                pageNumber: parseInt(page),
                pageSize: parseInt(size),
                sort: {
                    empty: false,
                    sorted: true,
                    unsorted: false
                },
                offset: startIndex,
                paged: true,
                unpaged: false
            },
            last: endIndex >= totalElements,
            totalElements: totalElements,
            totalPages: Math.ceil(totalElements / size),
            size: parseInt(size),
            number: parseInt(page),
            sort: {
                empty: false,
                sorted: true,
                unsorted: false
            },
            first: parseInt(page) === 0,
            numberOfElements: paginatedContent.length,
            empty: paginatedContent.length === 0
        };

        res.status(200).json(response);
    });
});

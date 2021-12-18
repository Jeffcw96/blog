db.createView('vw-user-expenses', 'user', [
            {
                $lookup: {
                    from: "expenses-category",
                    let: { "userRefId": "$ref_id" },
                    pipeline: [
                        {
                            $match: {
                                $expr: { $eq: ["$user_ref_id", "$$userRefId"] }
                            }
                        },
                        {
                            $lookup: {
                                from: "expenses",
                                let: { "categoryRefId": "$ref_id" },
                                pipeline: [
                                    {
                                        $match: {
                                            $expr: { $eq: ["$expenses_category_ref_id", "$$categoryRefId"] }
                                        }
                                    },
                                    {
                                        $project: {
                                            _id: 0,
                                            ref_id: 1,
                                            label: 1,
                                            value: 1
                                        }
                                    }
                                ],
                                as: "expenses"
                            },
                        },
                        {
                            $project: {
                                _id: 0,
                                __v: 0,
                                user_ref_id: 0
                            }
                        }
                    ],
                    as: "expenses_categories"
                }
            },
            {
                $project: {
                    _id: 0,
                    __v: 0
                }
            }
        ])